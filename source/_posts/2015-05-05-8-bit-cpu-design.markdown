---
layout: post
title: "8 bit CPU design"
date: 2015-05-05 14:02:05 +0800
comments: true
categories: ['VHDL']
---
Finally it works! 😂  
<!--more-->

Design pattern  
![design pattern](/images/posts/8bitcpu_design.png)

Source code of my CPU VHDL implementation on a Xilinx XC2S150 FPGA.

```VHDL 8bit_cpu.vhd
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.STD_LOGIC_ARITH.ALL;
use IEEE.STD_LOGIC_UNSIGNED.ALL;

-- ===================================================================================
-- ===================================================================================
entity cpu_8 is
    Port (
        DB:     inout std_logic_vector(7 downto 0);     --数据总线
        AB:     buffer std_logic_vector(15 downto 0);   --地址总线
        CI:     buffer std_logic_vector(31 downto 0);   --其中低10位是MPC的输出
        CO:     in std_logic_vector(31 downto 0);       --微指令输入到MIR
      
        --控制总线
        CLK, RESET, RUN:              in std_logic; 
        MWR, MRD, IOR, IOW, MCLK:     buffer std_logic;
        --外部观察
        CTRL1, CTRL2, CTRL3, CTRL4:   buffer std_logic;
        MUX:                          in std_logic_vector(2 downto 0);
        PRIX, KRIX:                   in std_logic
    );
end cpu_8;
-- ===================================================================================
-- ===================================================================================
architecture cpu_8_behave of cpu_8 is
--内部信号定义部分

--signal in M 存储器
    --读写控制脉冲
    signal CWR, CRD:        std_logic;
    --读写控制信号
    signal CWRX, CRDX:      std_logic;

--signal in A 累加器
    signal GA, ASR, CA:     std_logic;
    signal A:               std_logic_vector(7 downto 0);

--signal in ACT累加器暂存器
    signal ACT:             std_logic_vector(7 downto 0);
    signal GC, CC:          std_logic;

--signal in TMP 临时寄存器
    signal TMP:             std_logic_vector(7 downto 0);
    signal GT, CT:          std_logic;

--signal in Ri 寄存器
    signal R7, R6, R5, R4, R3, R2, R1, R0, ROUT:    std_logic_vector(7 downto 0);
    signal WRE, WRC:        std_logic;
    signal RS:              std_logic_vector(2 downto 0);

---多路选择器 MUXA, MUXB, MUXC, MUXD
--signal in MUXA
    signal MXA:             std_logic;

--signal in MUXB
    signal OB:              std_logic;
    signal MXB:             std_logic_vector(1 downto 0);

--signal in MUXC
    signal MXC:             std_logic_vector(1 downto 0);

--signal in MUXD
    signal PLD:             std_logic_vector(2 downto 0);
    signal PCADD:           std_logic;

--signal in MUXE
    signal MXE:             std_logic;
    signal CYA:             std_logic;
    signal CYD:             std_logic;

--signal in ALU
    signal FA, FB, FF:      std_logic_vector(8 downto 0);
    signal S:               std_logic_vector(2 downto 0);

--signal in CY
    signal CY, CP, CCK:     std_logic;

--signal in Z
    signal ZY:              std_logic;
    signal ZP:              std_logic;
    signal ZCK:             std_logic;

--signal in PC 
    signal PC:              std_logic_vector(15 downto 0);
    signal PCK:             std_logic;
    signal PINC, PRST:      std_logic;
    signal ADR:             std_logic_vector(15 downto 0);      --全地址
    
--signal in IR
    signal IR:              std_logic_vector(7 downto 0);

    signal CCI, GI:         std_logic;

--signal in ADRH
    signal ADRH:            std_logic_vector(7 downto 0);
    signal GA1, CA1, AHS:   std_logic;

--signal in ADRL
    signal ADRL:            std_logic_vector(7 downto 0);
    signal GA2, CA2:        std_logic;

--signal in MPC
    signal MPC, MD:         std_logic_vector(9 downto 0);
    signal MCLR, MPLD:      std_logic;
    signal MPCK:            std_logic;

--signal in SP
    signal SP:              std_logic_vector(15 downto 0);
    signal SSP:             std_logic_vector(1 downto 0);
    signal SCK:             std_logic;          

--signal in MIR
    signal MIR:             std_logic_vector(31 downto 0);
    signal MICK:            std_logic;

-- signal for Print and Key
    signal KOUT,POUT:       std_logic;

-- ==================================================================================
-- ==================================================================================

begin

--时钟信号与复位信号

---时钟信号
    pMCLK:
    process(MCLK, CLK, RESET, RUN)
    begin
    if((RUN = '0') or (RESET = '0')) then MCLK <= '0';
            elsif(CLK'event and CLK = '0') then MCLK <= not MCLK;
    end if;
    end process pMCLK;

    MPCK    <= not MCLK and CLK;
    MICK    <= not MPCK;
    
    WRC     <= MCLK;        --寄存器 Ri 时钟
    PCK     <= MCLK;        --PC 时钟
    CA      <= MCLK;        --A 时钟
    CT      <= MCLK;        --TMP 时钟
    CC      <= MCLK;        --ACT 时钟
    CCI     <= MCLK;        --IR 时钟
    CA1     <= MCLK;        --ADRH 时钟
    CA2     <= MCLK;        --ADRL 时钟
    CCK     <= MCLK;        --CY 时钟
    SCK     <= MCLK;        --SP 时钟
    ZCK     <= MCLK;        --ZY 时钟
    
    
----复位信号
    PRST   <= RESET; 
    
    pMCLR:
    process(MCLK, RESET)
    begin
        if(RESET = '0') then MCLR <= '0';
            elsif(MCLK'event and MCLK = '1') then MCLR <= RUN;
        end if;
    end process pMCLR;
    
    CWR     <= CWRX or not MCLK;
    CRD     <= CRDX or not MCLK;
    
    MRD     <= CRD or AB(15);
    MWR     <= CWR or AB(15) or not CLK;
    IOW     <= not AB(15) or not AB(1) or CWR or not CLK;
    IOR     <= not AB(15) or not AB(0) or CRD;
    
    --功能部件
    
----MPC的定义
    pMPC:
    process(MPLD, MPCK, MCLR)
    begin
        if(MCLR = '0') then 
            MPC <= "0000000000";
        elsif(MPCK'event and MPCK = '1') then
            if(MPLD = '0') then 
                MPC <= MD;
            else MPC <= MPC + 1;
            end if;
        end if;
    end process pMPC;
    
    CI(9 downto 0) <= MPC;
    
----为程序计数器入口MD的定义
    MD(0)           <= '1';
    MD(1)           <= '1';
    MD(2)           <= '1';
    MD(7 downto 3)  <= IR(7 downto 3);
    MD(9 downto 8)  <= "00";
    
----MIR的定义
    pMIR:
    process(MICK)
    begin
        if(MICK'event and MICK = '1') then 
            MIR <= CO;
        end if;
    end process pMIR;

----微程序控制信号
    CWRX    <= MIR(29);
    CRDX    <= MIR(28);
    MPLD    <= MIR(27);
    MXC(1)  <= MIR(26);
    MXC(0)  <= MIR(25);
    SSP(1)  <= MIR(24); 
    SSP(0)  <= MIR(23);
    PINC    <= MIR(22);
    PLD(2)  <= MIR(21); 
    PLD(1)  <= MIR(20); 
    PLD(0)  <= MIR(19); 
    MXA     <= MIR(18);
    S(2)    <= MIR(17);
    S(1)    <= MIR(16); 
    S(0)    <= MIR(15);
    MXE     <= MIR(14);
    CP      <= MIR(13);
    ZP      <= MIR(12);
    MXB(1)  <= MIR(11); 
    MXB(0)  <= MIR(10);
    OB      <= MIR(9); 
    GA2     <= MIR(8); 
    AHS     <= MIR(7); 
    GA1     <= MIR(6); 
    GI      <= MIR(5);
    GT      <= MIR(4); 
    GC      <= MIR(3); 
    ASR     <= MIR(2); 
    GA      <= MIR(1); 
    WRE     <= MIR(0);
    
----寄存器Ri的定义
    RS(2 downto 0) <= IR(2 downto 0);

    pRi:
    process(WRC, WRE, RS)
    begin
        if (WRC'event and WRC ='0') then 
            if (WRE = '0') then 
                case RS is
                    when "000" => R0 <= DB;
                    when "001" => R1 <= DB;
                    when "010" => R2 <= DB;
                    when "011" => R3 <= DB;
                    when "100" => R4 <= DB;
                    when "101" => R5 <= DB;
                    when "110" => R6 <= DB;
                    when "111" => R7 <= DB;
                    when others => NULL;
                end case;
            end if;
        end if;
    end process pRi;
    
----寄存器输出ROUT的定义
    ROUT    <=  R0 when RS = "000" else
                R1 when RS = "001" else
                R2 when RS = "010" else
                R3 when RS = "011" else
                R4 when RS = "100" else
                R5 when RS = "101" else
                R6 when RS = "110" else
                R7;
        
----MUXB的定义
    pMUXB:
    process(OB, MXB)
    begin
        if(OB = '0') then
            case MXB is
                when "00" => DB <= FF(7 downto 0);
                when "01" => DB <= PC(15 downto 8);
                when "10" => DB <= PC(7 downto 0);
                when others => DB <= ROUT;
            end case;
        else
            DB <="ZZZZZZZZ";
        end if;
    end process pMUXB;
    
----MUXC的定义
    AB <= PC                when MXC = "00" else
          ADRH&ADRL         when MXC = "01" else
         SP                 when MXC = "10";
    
----MUXD的定义
    PCADD   <=  '0'         when PLD = "000" else
                CY          when PLD = "001" else
                not ZY      when PLD = "010" else
                not KRIX    when PLD = "011" else
                not PRIX    when PLD = "100" else
                '1'         when PLD = "101" else
                ZY          when PLD = "110";
    
----TMP的定义
    pTMP:
    process(CT, GT)
    begin
        if(CT'event and CT = '0') then
            if(GT = '0') then 
                TMP <= DB;
            end if;
        end if;
    end process pTMP;

----A的定义    
    pA:
    process(CA, ASR)
    variable POOPOO:std_logic_vector(7 downto 0);
    begin
        if(CA'event and CA = '0') then
            if(ASR = '0') then 
                POOPOO := A;
                CYA  <= POOPOO(0);      --移出的末位
                A    <= POOPOO(7) & POOPOO(7 downto 1);
            elsif(GA = '0') then
                A <= DB;
            end if;
        end if;
    end process pA;
    
----MUXE的定义
     CYD <= FF(8) when MXE = '0' else
                CYA;
    
----ACT的定义
    pACT:
    process(CC, GC)
    begin
        if(CC'event and CC = '0') then
            if(GC = '0') then 
                ACT <= A;
            end if;
        end if;
    end process pACT;
    
    FA <= '0'&ACT; --扩展成九位
    FB <= '0'&ROUT when MXA = '0' else
          '0'&TMP;
    
----ALU的定义
    FF  <=  FA + FB         when S = "000" else
            FA - FB         when S = "001" else
            FA              when S = "010" else
            FB              when S = "011" else
            not FB          when S = "100" else
            "000000000"     when S = "101" else
            "000000000"     when S = "110" else
            FA + '1';
    
----CY的定义
    pCY:
    process(CCK, CP, FF)
    begin
        if(CCK'event and CCK = '0') then 
            if(CP = '0') then 
                CY <= CYD;
            end if;
        end if;
    end process pCY;
    
----ZY的定义
    pZY:
    process(ZCK, ZP, FF)
    begin
        if(ZCK'event and ZCK = '0') then 
            if(ZP = '0') then 
                if(FF = "000000000") then 
                    ZY <= '1';
                else
                    ZY <= '0';
                end if;
            end if;
        end if;
    end process pZY;
    
----PC的定义   
    pPC:
    process(PCK, PRST, PCADD)
    begin
        if(PRST = '0') then 
            PC <= "0000000000000000";
        elsif(PCK'event and PCK = '0') then
            if(PCADD = '1') then PC <= AB;
            elsif(PINC = '0') then 
                PC <= PC + 1;
            end if;
        end if;
    end process pPC;
    
----IR的定义
    pIR:
    process(CCI, GI, DB)
    begin
        if(CCI'event and CCI = '0') then 
            if(GI = '0') then 
                IR <= DB;
            end if;
        end if;
    end process pIR;
    
----ADRH的定义
    pADRH:
    process(CA1, GA1, AHS, DB)
    begin
        if(CA1'event and CA1 = '0') then 
            if(AHS = '0') then ADRH <= "01111110";
                elsif(GA1 = '0') then 
                    ADRH <= DB;
            end if;
        end if;
    end process pADRH;
    
----ADRL的定义
    pADRL:
    process(CA2, GA2, DB)
    begin
        if(CA2'event and CA2 = '0') then 
            if(GA2 = '0') then 
                ADRL <= DB;
            end if;
        end if;
    end process pADRL;
    
----SP的定义
    pSP:
    process(SCK, SSP)
    begin
        if(SCK'event and SCK = '0') then 
            case SSP is
                when "01" => SP <= SP - 1;
                when "10" => SP <= SP + 1;
                when "11" => SP <= "0111111111111111";
                when others => NULL;
            end case;
        end if;
    end process pSP;
    
----
    CI(31 downto 24)    <=  A                           when MUX = "000" else
                            PC(15 downto 8)             when MUX = "001" else
                            ADRH                        when MUX = "010" else
                            R0                          when MUX = "011" else
                            R2                          when MUX = "100" else
                            R4                          when MUX = "101" else
                            R6                          when MUX = "110" else
                            TMP;
                                
    CI(23 downto 16)    <=  IR                          when MUX = "000" else
                            PC(7 downto 0)              when MUX = "001" else
                            ADRL                        when MUX = "010" else
                            R1                          when MUX = "011" else
                            R3                          when MUX = "100" else
                            R5                          when MUX = "101" else
                            R7                          when MUX = "110" else
                            ACT;
                                
    CI(15 downto 12)    <=  SP(15 downto 12)            when MUX = "000" else
                            SP(11 downto 8)             when MUX = "001" else
                            SP(7 downto 4)              when MUX = "010" else
                            SP(3 downto 0)              when MUX = "011" else
                            MIR(17 downto 14);
                                
    CI(11)              <=  KRIX                        when MUX = "000" else
                            PRIX                        when MUX = "001" else
                            OB                          when MUX = "010" else
                            MPLD                        when MUX = "011" else
                            MIR(22);
                                
    CI(10)              <=  PRIX                        when MUX = "000" else
                            KRIX                        when MUX = "001" else
                            PINC                        when MUX = "010" else
                            GI                          when MUX = "011" else
                            CO(3)                       when MUX = "100" else
                            CO(0);
    
end cpu_8_behave ;
```

And here's my current instruction set.

```asm instructionSet.asm
-MOV
A,Ri
00000iii

-MOV
Ri,A
00001iii

-MOV
A,@Ri
00010iii

-MOV
@Ri,A
00011iii

-ADD
A,Ri
00100iii

-SUB
A,Ri
00101iii

-MOV
A,#data8
00110000
dddddddd

-MOV
Ri,#data8
00111iii
dddddddd

-LDA
addr
01000000
aaaaaaaa
aaaaaaaa

-STA
addr
01001000
aaaaaaaa
aaaaaaaa

-JC
addr
01010000
aaaaaaaa
aaaaaaaa

-JMP
addr
01011000
aaaaaaaa
aaaaaaaa

-JNKB
addr
01100000
aaaaaaaa
aaaaaaaa

-JNPB
addr
01101000
aaaaaaaa
aaaaaaaa

-CALL
addr
01110000
aaaaaaaa
aaaaaaaa

-RET
01111000

-RSP
10000000

-SUB
A,@Ri
10001iii

-ASR
A
10010000

-CLR
addr
10011000
aaaaaaaa
aaaaaaaa

-PUSH
Ri
10100iii

-POP
Ri
10101iii

-JZ
addr
10110000
aaaaaaaa
aaaaaaaa

-ADC
A,Ri
10111iii

-enddef
```

Below is the assembly code for testing 2-digit addition.

```asm test.asm
MOV R7,#0H

W1: 
 JNPB W1
 MOV A,#AH
 STA 8002H
W2:
 JNPB W2
 STA 8002H

L2:
 JNKB L2
 LDA 8001H
 MOV R2,A
L1:
 JNKB L1
 LDA 8001H
 MOV R1,A
WL1:
 JNPB WL1
 STA 8002H
WL2:
 JNPB WL2
 MOV A,R2
 MOV R0,#10H
 ADD A,R0
 STA 8002H
WADD:
 JNPB WADD
 MOV A,#10H
 STA 8002H
WADD1:
 JNPB WADD1
 MOV A,#AH
 STA 8002H
WADD2:
 JNPB WADD2
 STA 8002H

L4:
 JNKB L4
 LDA 8001H
 MOV R4,A
L3:
 JNKB L3
 LDA 8001H
 MOV R3,A
W3:
 JNPB W3
 STA 8002H
W4:
 JNPB W4
 MOV A,R4
 MOV R0,#10H
 ADD A,R0
 STA 8002H
WEQ:
 JNPB WEQ
 MOV A,#19H
 STA 8002H
WEQ1:
 JNPB WEQ1
 MOV A,#AH
 STA 8002H
WEQ2:
 JNPB WEQ2
 STA 8002H

 MOV A,R4
 ADD A,R2
 MOV R6,A

 MOV A,R1
 ADD A,R3
 MOV R5,A

 MOV R0, #AH
 SUB A,R0
 JC Z1

 MOV R5,A
 MOV R0,#1H
 MOV A,R6
 ADD A,R0
 MOV R6,A

Z1:
 MOV R0,#AH
 MOV A,R6
 SUB A,R0
 JC Z2

 MOV R6,A
 MOV R7,#1H
Z2: 
W5:
 JNPB W5
 MOV A,R5
 STA 8003H
W6:
 JNPB W6
 MOV A,R6
 STA 8003H
W7:
 JNPB W7
 MOV A,R7
 MOV R0,#10H
 ADD A,R0
 STA 8003H

LOOP:
 JMP LOOP
```

Demon of 2-digit addition  
<iframe width="420" height="315" src="http://www.youtube.com/embed/dikTQ-9l2s0" frameborder="0" allowfullscreen></iframe>
