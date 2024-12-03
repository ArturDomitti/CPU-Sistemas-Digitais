# SSC0108 - Prática Sistemas Digitais

[Projeto Final - CPU.](./cpu.pdf)

### ALUNOS

|        Nome                         |    NUSP   |       
|:-----------------------------------:|:---------:|  
|   Artur Domitti Camargo             |  15441661 |   
|   Lucas Mello Ciosaki       	      |  14591305 |   
|   Lucas Alves da Silva		         |  11819553  | 

A CPU realiza os seguintes comandos com os seguintes códigos:

|        Operação       |    Código   |       
|:---------------------:|:-----------:|  
|   ADD                 |  0000 |   
|   SUB       	        |  0001 |   
|   AND		              |  0010 |  
|   OR                  |  0011 |   
|   NOT       	        |  0100 |   
|   CMP		              |  0101 | 
|   LOAD                |  1001 |   
|   STORE       	      |  1010 |   
|   MOV		              |  1011 | 
|   IN                  |  1100 |   
|   OUT       	        |  1101 |  

Código da CPU em VHDL:
```
LIBRARY ieee;
USE ieee.std_logic_1164.all;
use ieee.std_logic_unsigned.all;
use ieee.std_logic_arith.all;


ENTITY cpu is
	PORT(
		Enter, Reset, Clk: in std_logic;
		Data: in std_logic_vector(7 downto 0);
		inputA, inputB, inputR: in std_logic;
		Cmd: in std_logic_vector(3 downto 0);
		Result: out std_logic_vector(7 downto 0);
		Overflow, Sinal, Zero: out std_logic
	);
end cpu;


Architecture Behavioral of cpu is
	
	type states is (readCmd, readX, readY, Decode, Execute);
	signal current_state : states := readCmd;
	signal next_state : states:= readCmd;
	signal IR: std_logic_vector(3 downto 0);
	signal A, B, R, Q, wMemory, rMemory: std_logic_vector(7 downto 0);
	signal clearA, clearB, clearR: std_logic;
	signal address: std_logic_vector(7 downto 0);
	signal enableAlu, enableMemory: std_logic;
	signal xCode, yCode: std_logic_vector(1 downto 0);

	signal X, Y: std_logic_vector(7 downto 0);
	
	component alu
		PORT(
			Enable: in std_logic;
			Clock: in std_logic;
			Cmd: in std_logic_vector(3 downto 0);
			A, B: in std_logic_vector(7 downto 0);
			R: out std_logic_vector(7 downto 0);
			Overflow: out std_logic;
			Sinal: out std_logic;
			Zero: out std_logic
		);
	end component;
	
	component ram256x8
		PORT
	(
		address		: IN STD_LOGIC_VECTOR (7 DOWNTO 0);
		clock		: IN STD_LOGIC  := '1';
		data		: IN STD_LOGIC_VECTOR (7 DOWNTO 0);
		wren		: IN STD_LOGIC ;
		q		: OUT STD_LOGIC_VECTOR (7 DOWNTO 0)
	);
	end component;
	
	
	begin
	
	

	ula : alu
		PORT MAP(
			Enable => enableAlu,
			Clock => Clk,
			Cmd => IR,
			A => X, 
			B => Y,
			R => Q,
			Overflow => Overflow,
			Sinal => Sinal,
			Zero => Zero
		);
		
	memory: ram256x8
		PORT MAP(
			address => address,
			clock => Clk,
			data => wMemory,
			wren	=> enableMemory,
			q	=> rMemory
		);
	
	process(Enter, Reset, Clk, Data)
	begin
		
		
		if(reset = '1') then
			next_state <= readCmd;
			current_state <= readCmd;
			enableAlu <= '0';
			enableMemory <= '0';
			xCode <= "00";
			yCode <= "00";
		end if;
		
		if(rising_edge(Clk)) then
		
		current_state <= next_state;
			
			case current_state is
				when readCmd =>
					
					IR <= Cmd;
					
					if(Enter = '1') then
						next_state <= readX;
					elsif(Enter = '0' and next_state /= readX) then
						next_state <= readCmd;
					end if;
					
				when readX =>
					
					
					if(inputA = '1') then
						X <= A;
						xCode <= "00";
					elsif(inputB = '1') then
						X <= B;
						xCode <= "01";
					elsif(inputR = '1') then
						X <= R;
						xCode <= "10";
					else
						X <= Data;
						xCode <= "11";
					end if;
					
					
					if(Enter = '1') then
						if(IR = "0000" or IR = "0001" or IR = "0010" or IR = "0011" or IR = "0101" or IR = "1001" or IR = "1010" or IR = "1011" or IR = "1100") then
							next_state <= readY;
						else
							next_state <= Decode;
						end if;
					elsif(Enter = '0' and (next_state = readX)) then
						next_state <= readX;
					end if;
					
					
				when readY =>
					
					if(inputA = '1') then
						Y <= A;
						yCode <= "00";
					elsif(inputB = '1') then
						Y <= B;
						yCode <= "01";
					elsif(inputR = '1') then
						Y <= R;
						yCode <= "10";
					else
						Y <= Data;
						yCode <= "11";
					end if;
					
					if(Enter = '1') then
						next_state <= Decode;
					
					elsif(Enter = '0' and next_state = readY) then
						next_state <= readY;
					end if;
					
				
				when Decode =>
					case IR is
						when "0000" =>
							enableAlu <= '1';
							R <= Q;
						when "0001" =>
							enableAlu <= '1';
							R <= Q;
						when "0010" =>
							enableAlu <= '1';
							R <= Q;
						when "0011" =>
							enableAlu <= '1';
							R <= Q;
						when "0100" =>
							enableAlu <= '1';
							R <= Q;
						when "0101" =>
							enableAlu <= '1';
							R <= Q;
						
						when "1001" =>
							address <= Y;
							
							if(xCode = "00") then
								A <= rMemory;
							elsif(xCode = "01") then
								B <= rMemory;
							elsif(xCode = "10") then
								R <= rMemory;
							end if;
						
						when "1010" =>
							address <= Y;
							wMemory <= X;
							enableMemory <= '1';
							
						when "1011" =>
							if(xCode = "00") then
								A <= Y;
							elsif(xCode = "01") then
								B <= Y;
							elsif(xCode = "10") then
								R <= Y;
							end if;
						
						when "1100" =>
							if(xCode = "00") then
								A <= Y;
							elsif(xCode = "01") then
								B <= Y;
							elsif(xCode = "10") then
								R <= Y;
							end if;
						
						when "1101" =>
							Result <= X;
						when others =>
						end case;
					next_state <= Execute;
					 
				when Execute =>
					case IR is
						when "0000" =>
							
							R <= Q;
						when "0001" =>
							
							R <= Q;
						when "0010" =>
							
							R <= Q;
						when "0011" =>
							
							R <= Q;
						when "0100" =>
							
							R <= Q;
						when "0101" =>
							
							R <= Q;
						
						when others =>
						end case;
					enableAlu <= '0';
					enableMemory <= '0';
					next_state <= readCmd;
				end case;
				
				
				
				
		end if;
	end process;


end Behavioral;
```

Código da ALU em VHDL:
```
LIBRARY ieee;
USE ieee.std_logic_1164.all;
use ieee.std_logic_unsigned.all;
use ieee.std_logic_arith.all;


ENTITY alu is
	PORT(
		Enable: in std_logic;
		Clock: in std_logic;
		Cmd: in std_logic_vector(3 downto 0);
		A, B: in std_logic_vector(7 downto 0);
		R: out std_logic_vector(7 downto 0);
		Overflow: out std_logic;
		Sinal: out std_logic;
		Zero: out std_logic
	);

end alu;

Architecture Behavioral of alu is
	
	signal unsigned_A: std_logic_vector(8 downto 0):= (others => '0');
	signal unsigned_B: std_logic_vector(8 downto 0):= (others => '0');
	signal unsigned_R: std_logic_vector(8 downto 0):= (others => '0');
	
	begin
	unsigned_A <= "000000000" + A;
	unsigned_B <= "000000000" + B;
	
	process(Clock, Enable, Cmd, unsigned_A, unsigned_B)
	begin
	if(rising_edge(Clock) and Enable = '1') then
		case Cmd is
			when "0000" =>
				
				unsigned_R <= unsigned_A + unsigned_B;
				
			when "0001" =>
				unsigned_R <= unsigned_A - unsigned_B;
				if unsigned_A < unsigned_B then 
					Sinal <= '1';
				else
					Sinal <= '0';
				end if;
					
			when "0010" =>
				unsigned_R <= unsigned_A AND unsigned_B;
			when  "0011" =>
				unsigned_R <= unsigned_A OR unsigned_B;
			when "0100" =>
				unsigned_R <= not(unsigned_A);
			when "0101" =>
				if(unsigned_A = unsigned_B) then
					unsigned_R <= "000000000";
				elsif(unsigned_A < unsigned_B) then
					unsigned_R <= "000000001";
				elsif(unsigned_A > unsigned_B) then
					unsigned_R <= "000000010";
				end if;
			when others =>
				unsigned_R <= unsigned_R;
		end case;
	end if;
	
	R <= unsigned_R(7 downto 0);
	Overflow <= unsigned_R(8);
	
	end process;
	
	process(unsigned_R)
	begin
		if unsigned_R = "000000000" then 
			Zero <= '1';
		else
			Zero <= '0';
		end if;
	end process;
	
end Behavioral;
```

Código da memória 256x8 criada pela ferramenta do QUARTUS:
```
LIBRARY ieee;
USE ieee.std_logic_1164.all;

LIBRARY altera_mf;
USE altera_mf.altera_mf_components.all;

ENTITY ram256x8 IS
	PORT
	(
		address		: IN STD_LOGIC_VECTOR (7 DOWNTO 0);
		clock		: IN STD_LOGIC  := '1';
		data		: IN STD_LOGIC_VECTOR (7 DOWNTO 0);
		wren		: IN STD_LOGIC ;
		q		: OUT STD_LOGIC_VECTOR (7 DOWNTO 0)
	);
END ram256x8;


ARCHITECTURE SYN OF ram256x8 IS

	SIGNAL sub_wire0	: STD_LOGIC_VECTOR (7 DOWNTO 0);

BEGIN
	q    <= sub_wire0(7 DOWNTO 0);

	altsyncram_component : altsyncram
	GENERIC MAP (
		clock_enable_input_a => "BYPASS",
		clock_enable_output_a => "BYPASS",
		intended_device_family => "Cyclone IV E",
		lpm_hint => "ENABLE_RUNTIME_MOD=NO",
		lpm_type => "altsyncram",
		numwords_a => 256,
		operation_mode => "SINGLE_PORT",
		outdata_aclr_a => "NONE",
		outdata_reg_a => "UNREGISTERED",
		power_up_uninitialized => "FALSE",
		ram_block_type => "M9K",
		read_during_write_mode_port_a => "NEW_DATA_NO_NBE_READ",
		widthad_a => 8,
		width_a => 8,
		width_byteena_a => 1
	)
	PORT MAP (
		address_a => address,
		clock0 => clock,
		data_a => data,
		wren_a => wren,
		q_a => sub_wire0
	);



END SYN;
```
