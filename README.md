# lab08_cs350
Example from Lab08 Computer Organization &amp; Assembly Language 
// *** Lab01_VienYadavongsy_Sec 01 **

// CS 350, Spring 2015
// Lab 8: LC3 Simulator
// Illinois Institute of Technology, (c) 2014, James Sasaki

#include <stdio.h>
#include <stdlib.h> // for error exit()

// CPU Declarations -- a CPU is a structure with fields for the
// different parts of the CPU.
//
	typedef short int Word;          // type that represents a word of LC3 memory
	typedef unsigned short Address;   // type that represents an LC3 address

	#define MEMLEN 65536
	#define NREG 8

	typedef struct {
		Word mem[MEMLEN];
		Word reg[NREG];      // Note: "register" is a reserved word
		Address pc;          // Program Counter
		int running;         // running = 1 iff CPU is executing instructions
		Word ir;             // Instruction Register
		int instr_sign;      //   sign of instruction
    int opcode;          //   opcode field
		int reg_R;           //   register field
		int addr_MM;         //   memory field
		int cc;				      //   NEW: CONDITION CODE
	} CPU;

// Prototypes [note the functions are also declared in this order

	int main(int argc, char *argv[]);
	void initialize_control_unit(CPU *cpu);
	void initialize_memory(int argc, char *argv[], CPU *cpu);
	FILE *get_datafile(int argc, char *argv[]);

	void dump_control_unit(CPU *cpu);
	void dump_memory(CPU *cpu);
	void dump_registers(CPU *cpu);

	int read_execute_command(CPU *cpu);
	int execute_command(char cmd_char, CPU *cpu);
	void help_message(void);
	void many_instruction_cycles(int nbr_cycles, CPU *cpu);
	void one_instruction_cycle(CPU *cpu);
  void exec_HLT(CPU *cpu);

// defined the type then put in your arbitrary variable name
// Main program: Initialize the cpu, read initial memory values,
// and execute the read-in program starting at location 00.
//
int main(int argc, char *argv[]) {
	printf("LC3 Simulator skeleton: CS 350 Lab 8\n Vien Yadavongsy\n");
	CPU cpu_value, *cpu = &cpu_value;
	//get_datafile (argc, argv); ***
	initialize_control_unit(cpu);
	initialize_memory(argc, argv, cpu);
	dump_control_unit(cpu);
	dump_memory(cpu);

	char *prompt = "> ";
	printf("\nBeginning execution; type h for help\n%s", prompt);

	int done = read_execute_command(cpu);
	while (!done) {
		printf("%s", prompt);
		done = read_execute_command(cpu);
	}
	return 0;
}
// Initialize the control registers (pc, ir, running flag) and
// the general-purpose registers
//
void initialize_control_unit(CPU *cpu) {
	cpu -> running = 1;   // this  how initialize control unit
	cpu -> pc = 0;   // reg is our word type
	cpu -> ir = 0;
	cpu -> cc = 0; 

	for (int i = 0; i < NREG; i++)
    {
        cpu -> reg[i] = 0;  // needs be initialized  
		//cpu -> mem[i] = 0;  // thinking need this instead
    }
	printf("\nInitial control unit:\n");
	//dump_control_unit(cpu);
	printf("\n");
}

// Read and dump initial values for memory
//
void initialize_memory(int argc, char *argv[], CPU *cpu) {
	FILE *datafile = get_datafile(argc, argv);

	// Will read the next line (words_read = 1 if it started
	// with a memory value). Will set memory location loc to
	// value_read
	//
	for (int i = 0; i < MEMLEN; i++) {
		cpu -> mem[i] = 0;
	}

	Address loc = 0;   // have loc variable, the type changed (is US)
	Word value_read; // changed the type
	int words_read, done = 0;  

	// Each getline automatically reallocates buffer and
	// updates buffer_len so that we can read in the whole line
	// of input. bytes_read is 0 at end-of-file.  Note we must
	// free the buffer once we're done with this file.
	//
	// See linux command man 3 getline for details.
	//
	char *buffer = NULL;
	size_t buffer_len = 0, bytes_read = 0;

	//
	bytes_read = getline(&buffer, &buffer_len, datafile);
	words_read = sscanf(buffer, "%hx", &value_read);
		if (words_read > 0) {
			loc = value_read;
			cpu-> pc = value_read; 
		}
		else {
			printf("Error read file\n");
			exit(1);
		}
	bytes_read = getline(&buffer, &buffer_len, datafile);
	while (bytes_read != -1 && !done) {
		// If the line of input begins with an integer, treat
		// it as the memory value to read in.  Ignore junk
		// after the number and ignore blank lines and lines
		// that don't begin with a number.
		//
		words_read = sscanf(buffer, "%hx", &value_read);

		// ****STUB**** set memory value at current location to
		// value_read and increment location. Exceptions: If loc
		// is out of range, complain and quit the loop. If value_read
		// is outside -9999...9999m then it;s a sentinel and we should say so
		// and quit the loop

		if (words_read > 0)  {
				cpu -> mem[loc] = value_read;
				loc++;		
		}
		// GET next line and continue the loop
		bytes_read = getline(&buffer, &buffer_len, datafile);
	}

	free(buffer);  // return buffer to OS
	//dump_memory(cpu);
}

// Get the data file to initialize memory with.  If it was
// specified on the command line as argv[1], use that file
// otherwise use default.sdc.  If file opening fails, complain
// and terminate program execution with an error.
// See linux command man 3 exit for details.
//
FILE *get_datafile(int argc, char *argv[]) {
	char *default_datafile_name = "default.hex";
	char *datafile_name;  // variable: datafile_name
        // *datafile_name = *default_datafile_name;  ----see below
        if (argc > 1 )  // note to self: J. lab 3
            datafile_name = argv[1];
        else {
            *datafile_name = *default_datafile_name;  // should this be: = argv[1]
        }
        printf("Intiializing memory from %s \n", datafile_name); // notice this change

        FILE *datafile = fopen(datafile_name, "r");

        //********STUB********* if the open failed, complain and call
        // exit (EXIT_FAILURE); to quit the entire program

	 if (datafile == NULL) {
        printf("%s is not a valid file!\n", datafile_name);
        exit(EXIT_FAILURE);
    }
    return datafile;
}
// dump_control_unit(CPU *cpu): Print out the control and
// general-purpose registers
//
void dump_control_unit(CPU *cpu) {  
	char cc_value; 
	if (cpu -> cc == 0) {
		cc_value = 'Z'; 
	}
	else 
		if (cpu -> cc > 0) {
			cc_value = 'P'; 
		}
	printf ("PC:     %02hx    IR :    %04hx    CC: %c 	RUNNING   : %d\n", cpu -> pc, cpu -> ir, cc_value, cpu -> running);
	dump_registers(cpu);
	printf ("\n");
}
 
void dump_memory(CPU *cpu) {
		printf ("\n");
		int loc; 
        for (loc = 0; loc < MEMLEN; loc++) {
			if (cpu->mem[loc] != 0) 
			printf("Location: %hx,    %hx    %d \n", loc, cpu->mem[loc], cpu->mem[loc]);
      }
		//dump_registers(CPU *cpu); 
}
void dump_registers(CPU *cpu) {   
// print all registers here 
        for (int i = 0; i < NREG; i ++)
        {
            if (i == 4) // random 
				printf ("\nR%d : %5hx %5hx \n", i, cpu -> reg[i], cpu ->reg[i]);
			else
				printf ("\nR%d : %5hx %5hx \n", i, cpu -> reg[i], cpu ->reg[i]); 
        }
		//dump_registers(cpu -> reg_R); 
        printf ("\n");
}


// Read a simulator command from the keyboard ("h", "?", "d", number,
// or empty line) and execute it.  Return true if we hit end-of-input
// or execute_command told us to quit.  Otherwise return false.
//
int read_execute_command(CPU *cpu) {  
	// Buffer for the command line from the keyboard, plus its size
	//
	char *cmd_buffer = NULL;
	size_t cmd_buffer_len = 0, bytes_read = 0;

	// Values read using sscanf of command line
	//
	int nbr_cycles;
	char cmd_char;
	size_t words_read;	// number of items read by sscanf call

	int done = 0;	// Should simulator stop?

	bytes_read = getline(&cmd_buffer, &cmd_buffer_len, stdin);
	if (bytes_read == -1) {
		done = 1;   // Hit end of file
	}

	words_read = sscanf(cmd_buffer, "%d", &nbr_cycles);
	//***STUB*** If we found a number,do that many instruction cycles.
	// Otherwise sscanf for a character and call execute_command with it.
	// (Note the character might be '\n')
	// printf ("%s (%zu) num: %d, cmd_buffer, words_read, nbr_cycles);
	// printf ("\ndone!");

        if (words_read == 0)  {
            words_read = sscanf (cmd_buffer, "%c", &cmd_char); // too many: J.
            done = execute_command(cmd_char, cpu);
        }
        else {
             many_instruction_cycles(nbr_cycles, cpu);
         }

	free(cmd_buffer);
	return done;
}

// Execute a nonnumeric command; complain if it's not 'h', '?', 'd', 'q' or '\n'
// Return true for the q command, false otherwise
//
int execute_command(char cmd_char, CPU *cpu) {
	if (cmd_char == '?' || cmd_char == 'h') {
		help_message();
	}
	else if (cmd_char == 'q')  {
        printf(" Quitting \n");
        return 1;
    }
    else if (cmd_char == 'd')  {
        dump_control_unit(cpu);
        dump_memory(cpu);  
    }
    else if (cmd_char == '\n')  {
        one_instruction_cycle (cpu); 
    }
    else  {
        printf ("%c not a valid command! \n", cmd_char);
    }
	return 0;
}

// Print standard message for simulator help command ('h' or '?')
//
void help_message(void) {  
    printf("h or ? for help (prints this message \n");
    printf("d to dump the control unit and memory \n");
	  printf("q to quit \n");
 /*   printf ("An integer > 0 to execute that many instruction cycles \n");
    printf ("Press return, which executions one instruction cycle \n");
    printf ("SDC Instruction set: \n");
    printf ("0xxx: HALT \n");
    printf ("1RMM: Load Reg [R] <- M[MM]\n");
    printf ("2RMM: Store M[MM] <- Reg [R] \n");
    printf ("3RMM: Add M[MM] to Reg [R] (Reg [R] += M[MM]\n");
    printf("4Rxx: Reg[R] <- (=Reg[R]) {set Reg [R] to its arithmetic negative}\n");
    printf("5RMM: Load Immediate - Reg [R] <- sign * [MM]\n");
    printf("6RMM: Add Immediate - Reg [R] += sign * [MM] {add MM to R\n");
    printf("7xMM: BRANCH  to MM, ignore R - PC <- [MM]\n");
    printf("8RMM: BRANCH CONDITIONAL if sign (g [R]) = sign then PC <- [MM]\n");
    printf("90xx: GETC Read a character & copy ASCII representation into R0 - Reg [0] <- keyboard\n");
    printf ("91xx: OUT print character whose ASCII representation is in R0 - Print Char(Reg[0]\n");
    printf("92MM:  PUTS print a string at MM locations, stop when get to 0 - temp <- MM while Mem[temp] != 0 Print Mem[temp]; temp++ \n");
    printf("93MM: DMP print out values of control unit registers - Dump Control Unit\n");
    printf("94MM: MEM print values in MM for 10x10 table - Dump Memory\n");   */
}

// Execute a number of instruction cycles.  Exceptions: If the
// number of cycles is <= 0, complain and return; if the CPU is
// not running, say so and return; if the number of cycles is
// insanely large, complain and substitute a saner limit.
//
// If, as we execute the many cycles, the CPU stops running,
// then return.
//
void many_instruction_cycles(int nbr_cycles, CPU *cpu) {  
    int max_cycle = 100;
    if (nbr_cycles < 1){
        printf("Number of cycles should be > 0\n");
        return;
    }
    if ((cpu -> running) == 0) {
        printf ("CPU has been halted\n");
        return;
    }
    if (nbr_cycles > max_cycle ){
        printf("%d can not be done. Consider %d instead\n", nbr_cycles, max_cycle);
        nbr_cycles = max_cycle;
    }
    int cycle_left = nbr_cycles; 
    while  ((cpu -> running) && cycle_left > 0)  {
            one_instruction_cycle(cpu);
            cycle_left--;
    }
}

// Execute one instruction cycle
//
void one_instruction_cycle(CPU *cpu) {
	// If the CPU isn't running, say so and return.
	// If the pc is out of range, complain and stop running the CPU.
	//
	if ((cpu -> running) == 0) {
        printf("CPU has been halted\n");
	}
	// Get instruction and increment pc
	//
	int instr_loc = cpu -> pc;  // Instruction's location (pc before increment)
	(cpu -> ir) = (cpu -> mem[cpu -> pc++]);

	// Decode instruction into opcode, reg_R, addr_MM, and instruction sign
	// refer to the original struct revised: J>

    if ((cpu -> ir) < 0) {
        cpu -> instr_sign = -1;
        cpu -> ir *= -1;
    }
    else {
        cpu -> instr_sign = 1;
        cpu -> addr_MM = cpu -> ir%100;
        cpu -> reg_R = ((cpu -> ir / 100)%10);
        cpu -> opcode = ((cpu-> ir / 1000)% 10);
    }
	// Echo instruction  means to read the instruction back to user: the printf lines does this.
	//
	printf("At %02d instr %d %d %02d: ", instr_loc, (cpu -> opcode * cpu -> instr_sign), cpu -> reg_R, cpu -> addr_MM);

	char *cmd_buffer = NULL;
	char cmd_char;
	size_t cmd_buffer_len = 0;
	int i = 0;

	switch (cpu -> opcode) {
	case 0:
	    exec_HLT(cpu);
	    printf ("HALT\n");
	    break;
    case 1:
        (cpu -> reg[cpu -> reg_R]) = (cpu -> mem [cpu -> addr_MM]);
        printf("LOAD R%d <-  M[%d] = %d\n", cpu -> reg_R, cpu -> addr_MM,cpu -> reg[cpu -> reg_R]);
        break;
    case 2:
        (cpu -> mem[cpu -> addr_MM]) = (cpu -> reg [cpu -> reg_R]);
        printf ("STORE M[%d] <- R%d = %d\n", cpu -> addr_MM, cpu -> reg_R, cpu -> reg[cpu -> reg_R]);
        break;
    case 3:
        (cpu -> reg[cpu -> reg_R]) += (cpu -> mem[cpu -> addr_MM]);
        printf ("ADD R%d <- R%d + M[%d] = %d + %d = %d\n", cpu -> reg_R, cpu -> reg_R,
                cpu -> addr_MM,cpu -> reg[cpu -> reg_R], cpu -> mem[cpu-> addr_MM]);
        break;
    case 4:
        (cpu -> reg[cpu -> reg_R]) *= -1;
        printf ("NEGATE R%d <- -R%d = %d\n", cpu -> reg_R, cpu -> reg_R, cpu -> reg[cpu -> reg_R]);
        break;
    case 5:
        (cpu -> reg [cpu -> reg_R]) = cpu -> addr_MM * cpu -> instr_sign;
        printf("LDM R%d <- %d\n", cpu -> reg_R, cpu -> reg[cpu -> reg_R]);
        break;
    case 6:
        cpu -> reg [cpu -> reg_R] += cpu -> instr_sign * cpu -> addr_MM;
        printf ("ADDM R%d <- R%d + %d = %d = %d + %d = %d\n", cpu -> reg_R, cpu -> reg_R,
                (cpu -> addr_MM * cpu -> instr_sign), cpu -> reg [cpu -> reg_R], (cpu -> addr_MM * cpu -> instr_sign));
        break;
    case 7:
        ((cpu -> pc) = (cpu -> addr_MM));
        printf ("BRANCH %d\n", cpu -> addr_MM);
        break;
    case 8:
        if (cpu -> instr_sign == -1) 
            printf ("BRANCH CONDITIONAL %d if R%d = %d > 0: Yes\n", cpu -> addr_MM, cpu -> reg_R, cpu -> reg[cpu -> reg_R]);

        else
            printf ("BRANCH CONDITIONAL %d if R%d = %d > 0: No\n", cpu -> addr_MM, cpu -> reg_R, cpu -> reg[cpu -> reg_R]);

        if (cpu -> reg[cpu -> reg_R] * cpu -> instr_sign > 0)   {
            printf ("Yes");
            ((cpu -> pc) = (cpu ->addr_MM));
        }
        break; 
	
    case 9: if (cpu -> reg_R) {
        printf ("I/O Read char\n");
        printf ("Enter a char (and/or press return): ");
        getline (&cmd_buffer, &cmd_buffer_len, stdin);
        sscanf(cmd_buffer, "%c", &cmd_char);
        cpu -> reg[0] = cmd_char;
        printf ("R0 <- %d\n", cpu -> reg[0]);
		}
		break;

    case 90:
        printf ("I/O 1: Print char in R0 (= %d): %c", cpu -> reg[0], cpu -> reg[0]);
        break;
    case 92:
        printf ("I/O 2: Print String: ");
        i = cpu -> addr_MM;
        while (i < MEMLEN && cpu -> mem[i] != 0)  {
            printf ("%c", cpu -> mem[i]);
            i++;
        }
        break;
    case 93:
        printf ("I/O 3: Dump Control Unit: \n");
        dump_control_unit(cpu);
        break;
    case 94:
        printf ("I/O 4: Dump Memory: \n");
        dump_memory(cpu);
		break; 
	default: printf ("Bad opcode !??! %d \n", cpu -> opcode);
		break; 
	}
    printf ("\n");

}
// Execute the halt instruction (make CPU stop running)
//
void exec_HLT(CPU *cpu) {
	printf("HALT\nHalting\n");
	cpu -> running = 0;
} 

