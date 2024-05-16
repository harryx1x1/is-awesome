Source is: https://www.youtube.com/watch?v=DcLQ3daqnqQ

Notes:
- jolt's new staff: replace execution with lookup(with lasso)
- cpu tasks: 
	- fetch, decode and execute instructions
	- read/write RAM
- riscv cpu
	- has 32 registers, register is not so efficient in zkvm, so no not need to differentiate it with RAM
- fetch: jolt handle RAM with offline memory checking techniques(with sumcheck)
- decode: with R1CS. 
	- per step of cpu
		- about 40 constraints
		- about 80 witness elements committed 
	- if you fetch program counter from RAM, you have to decompose it into different fields, so we need to make sure the decomposition is right
- main differences from pre-existing zkvm:
	- use sumcheck everywhere, to minimize the amount of data committed by prover
	- everything is offline memory checking other than the R1CS