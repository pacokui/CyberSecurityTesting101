1. Screenshot of AFL++ Execution
Insert your screenshot below:
![AFL++ Execution](./path-to-your-image.png)
2. Required Commands to Complete This Task
1. Install AFL++
sudo apt update && sudo apt install afl++
2. Compile the Binary with AddressSanitizer
AFL_USE_ASAN=1 afl-cc -o sample sample.c
3. Create Input/Output Directories and Seed File
mkdir inputs outputs
echo "initial test string" > inputs/seed.txt
4. Run AFL++
afl-fuzz -i inputs -o outputs -- ./sample @@
5. Reproduce the Crash to View ASAN Output
./sample outputs/default/crashes/id:000000*
3. AddressSanitizer Output Analysis
Does it identify the line of code?
Yes.
The stack trace shows:
main ... sample.c:16:9
This indicates execution reached line 16 in sample.c.
It also identifies that the overflowed variable is buffer, originally declared on line 6.
What other information does it provide?
Vulnerability type: stack-buffer-overflow
Violation: WRITE of size 61
Buffer size: 50 bytes
Overflow amount: 11 bytes
Provides a shadow memory map
Shows write into protected redzone memory
This confirms both the vulnerability type and memory corruption details.
4. Crash Statistics
Total crashes found
2560
Unique crashes
1
AFL++ filters duplicate crashes automatically. The saved crashes count represents unique vulnerabilities.
5. Cycles Information
Cycles done
536
What does "cycles" mean?
A cycle means AFL++ processed every test case in its queue and applied its full mutation strategy once.
In this case, the queue was processed 536 times.
6. When Should You Stop the Fuzzer?
You should stop when:
Cycles indicator turns green
No new paths discovered for a long time
No new crashes found for hours/days
Code coverage stops increasing
At that point, the fuzzer likely exhausted meaningful mutations.
7. Current Fuzzing Strategy
Strategy
havoc
The havoc stage applies aggressive random mutation combinations such as:
Bit flips
Byte overwrites
Insertions
Deletions
Stacked mutations
It is designed to generate chaotic input variations.
