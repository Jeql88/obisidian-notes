## October 2025 Exam
###### **Q1**
![[Pasted image 20260407205831.png]]

**thought process:**
A = {1, 2 ,3}  B = {1, 2, 5}
A U B = {1, 2, 3, 5}
A & B = {1, 2}
Complement = {3, 5}
therefore not a.
not b cause A U B has more than in A
**Answer: C** 

###### **Q2.** Which of the following is the binary number that is obtained by adding the binary  numbers 01011010 and 01101011? Here, the binary numbers are expressed as positive 8-bit values.
a) 00110001 b) 01111011 c) 10000100 d) 11000101

**thought process:**
01011010
01101011 +
____________
11000101
**Answer: D**

###### **Q3**. Which of the following is the combination of the mean and median of the data?
[Data]
10, 20, 20, 20, 40, 50, 100, 440, 2000
Mean Median
a) 20 40
b) 40 20
c) 300 20
d) 300 40

**thought process:**
10 + 20 + 20 + 20 + 40 + 50 + 100 + 440 + 2000 
nevermind, by process of elimination, when median is definitely 40

**Answer: D**

###### **Q4.** Which of the following is the appropriate description concerning the prefix of the International System of Units (SI) that is used to indicate CPU clock frequency, communication speed, and other such things?
a) G multiplied by 10 to the power of 6 is T.
b) M multiplied by 10 to the power of 3 is G.
c) M multiplied by 10 to the power of 6 is G.
d) T multiplied by 10 to the power of 3 is G.

 **thought process:**
 need to search regarding this topic
 search results:
 ![[Pasted image 20260407210842.png]]
a - G * 10^6 = 10^15 so wrong
b - M * 10^3 = 10^9 - correct

**Answer: B**

###### **Q5.** There is a desk that can hold at most four (4) files. On this desk, six (6) files A through F are used for a job. When the fifth file needs to be put on the desk, the file with the longest time since the last use among the four (4) will be put in a drawer. If the files are put on the desk and referenced in order of A, B, C, D, E, C, B, D, F, B, which of the following is the last file to be put in the drawer?
a) A b) B c) D d) E

 **thought process:**
 ~~desk - 4 files max~~
 ~~6 files - A through F are used for a job~~
 ~~if 5th file - file with longest time last used among 4 in desk will be put in drawer~~
 ~~A B C D~~ 
 ~~(A put in drawer) B C D E~~
 ~~look at last 4 file letters, then the letter that was last removed is the one to the left of it~~
~~**Answer: B**~~

must actually update the usage! not readd
[A, B, C, D] - must add E
[B, D, E, C] - must add C, but C exists, therefore C becomes "most recent"
[D, E, C, B] - must add B, but B exists, therefore B becomes "most recent"
[E, C, B, D] - must add D, but D exists, therefore D becomes "most recent"
[C, B, D, F] - must add F, store E
[C, D, F, B] - must add Bm but B exists, therefore B becomes "most recent"
Last file to be placed on drawer is E
**Answer: D**

###### Q6. At a certain company, employees get a medical allowance equal to 5% of their monthly base salary in addition to their monthly base salary. If the total of the monthly base salary and medical allowance is more than 5,000, a 20% tax is deducted from the total. In the program below, the function calculateTotalSal receives the monthly base salary as an argument and returns the monthly total salary after tax deduction. Which of the following is the correct combination of pieces of code to be inserted into ___A___ and ___B___ in the program? Here, the monthly base salary is a multiple of 100.
![[Pasted image 20260408000503.png|458]]
 **thought process:**
 employees medical allowance = 5% of monthly base salary
 if monthly base salary + medical allowance is > 5000
 then 20% tax is deducted from total.
 5000
 0.8
 **Answer: B**
 
###### Q7. Which of the following is the appropriate way for extracting data from a stack that stores multiple data?
a) Data is extracted from an arbitrary location that is specified, regardless of the sequence
that data is stored in.
b) Data that was stored last is extracted first.
c) Data that was stored first is extracted first.
d) Data has a key, and it is extracted according to the priority of the key

 **thought process:**
 stack is first in first out LIFO
  **Answer: B**

###### Q8. In the program below, the function subTotal receives an integer array inputarr as an argument, returns an array arr in which the sum of all elements of original array inputarr up to that location is stored in the same location. Which of the following is the correct combination of pieces of code to be inserted into ___A___ and ___B___ in the program. Here, the size of the array inputarr is greater than 1, and elements outside the bounds of the array must not be accessed, and the array index starts at 1.
![[Pasted image 20260408001114.png|390]]
 **thought process:**
 subTotal receives integer array inputarr as an arg and returns arr where sum of all elements of original array up to that location is stored in the same location
 if inputarray [1,2,3,4,5]
 arr[1] = inputarr[1]  -> arr = [1]
 arr[1] = inputarr[1] + arr[i - 1]
 arr  = [1]
 arr[2] =  inputarr[2] + arr[i-1] = 2 + 1 therefore arr = 3, correct
  ~~**Answer: B**~~
 actually, must consider that we skip the first one. next time read for details like that more attentively
 **Answer: D**

###### Q9. Which of the following is the explanation of a characteristic of hypertext?
a) It has a function for the creation and editing of a range of expressions.
b) It has a function for the creation and editing of a range of shapes.
c) It provides a range of templates, and enables their use.
d) It has a mechanism that enables relevant information to be accessed by embedding a link
in any position in a text.
 **thought process:**
 pretty sure its D based from previous stock knowledge
  **Answer: D**
###### Q10. Which of the following is designed to virtually reduce the time for the CPU to access the main memory, in order to improve the processing efficiency of a PC?
a) SSD b) Virtual memory
c) Cache memory d) Defragmentation
 **thought process:**
 says virtually, therefore not SSD. Virtual memory is too big. Not defragmentation for sure, most likely Cache
 **Answer: C**
 
###### Q11. Which of the following is the most appropriate description concerning a compatible CPU?
a) It can run the same OS or the application software that can be run on the original CPU.
b) It must not be developed nor manufactured as long as the patents of the original CPU are
valid.
c) A compatible CPU for single-core CPU has been developed, while one for multi-core
CPU does not exist.
d) It is a CPU that is intended to improve the performance of an outdated PC, and is not
adopted for a new model of PC.
 **thought process:**
 Definitely not C, not D either. B also doesn't seem like it. Most likely answer is A
  **Answer: A**
  
###### Q12. In the description below concerning the connection between a PC and a peripheral device, which of the following is the appropriate combination of words to be inserted into blanks A and B?
![[Pasted image 20260408002240.png]]
 **thought process:**
 connection between PC and peripheral device. 
 definitely a or b, plug and play function
**Answer: A**

###### Q13. There is an IoT system that automatically opens and closes the water gate for a rice field by measuring the water level in the rice field. Which of the following is the appropriate combination of terms to be inserted into blanks A and B in the figure?

 ![[Pasted image 20260408002452.png|510]]
 **thought process:**
 IoT system that closes and opens water gate for a rice field based on water level in the rice field.
 most likely A is sensor since we are detecting the water level in the rice field. ~~B most likely C cause IoT gateway is mentioned twice~~
 ~~**Answer: C**~~
 Actuator - does the action like closing the field.
 IoT gateway is the along the levels of the server that analyzes the received data from the Sensor
 **Answer: D**
 
###### Q14. The system configurations A through C are made up by connecting multiple identical devices. Which of the following is the list where A through C are sorted in descending order of availability? Here, represents a device. The parallel connection requires only one (1) of the devices to be functional, while the series connection requires all devices to be functional.
![[Pasted image 20260410230051.png|512]]
**thought process:**
C has more chances to go through, then B has more chances to go through after the initial so i think d.
**Answer: D**

###### Q15. When the processing speed of the system overall is constrained because of slow processing speed in one of the components, which of the following is the term for the component that is causing the problem?

a) Throughput b) Defragmentation
c) Flowchart d) Bottleneck

**thought process:**
slow because of one of the components.
not a or c for sure. not b.
**Answer: D**

###### Q16. Among the descriptions A through C below concerning the effects that are obtained by simultaneously running multiple virtual servers on one (1) physical computer, which of the following lists all and only the appropriate descriptions?

A: Each virtual server can run an OS with a different version, and the resources of the
physical computer can be utilized effectively.
B: The throughput that is gained when the number of virtual servers is increased is the
same as when the number of physical computers is increased by the same number.
C: A volume of data that is the same as the capacity of the HDD on the physical computer
can be simultaneously recorded on all of the virtual servers.

a) A b) A, C c) B d) C

**thought process:**
A - most likely true, b not
C - doesn't seem that true
Answer: A

###### Q17. Which of the following is the appropriate explanation of a deadlock?
a) It is a state in which the processes of a computer are prohibited from getting any access
including accidental access to information that must not be accessed.
b) It is a state in which, if the login attempt of a user fails more than a predefined number
of times, the user is prohibited from logging in to the computer for a predefined time
period or until the system administrator resets the user account.
c) It is a state in which, when common resources are exclusively used, two or more
processes are endlessly waiting for each other to release the resources they are
occupying.
d) It is a state in which processes in the ready state under a multi-programming
environment have used up the CPU time allocated by the OS.

**thought process:**
a - computers can't get access including accidental access to information that must be accessed
b - i think not
c - i think this
d -  i think not
overall, i think based on the wording, C matches it the most.
Answer: C

Q18. In a web server, five (5) directories have a hierarchical structure as shown in the figure below. In an HTML document stored in directory B, which of the following references the file img.jpg stored in the directory E? Here, directories and files are referenced using the method described below.
![[Pasted image 20260410231729.png]]
a) ../A/D/E/img.jpg b) ../D/E/img.jpg
c) ./A/D/E/img.jpg d) ./D/E/img.jpg
**thought process:**
its b because we are in directory B so we must go back to A first, then go to D then E

Answer: B

Q19. There is a system where a full backup is made after closing time every Sunday, and an incremental backup is made after closing time on Monday through Saturday. On a Wednesday, a failure happened during business hours, so a decision was made to restore data to the state at the closing time on Tuesday by using backup files. Which of the following lists all and only the necessary backup files for restoring data? Here, an incremental backup file means a backup file that contains only the data that has been modified since the previous backup (a full backup or an incremental backup) was made.
a) A full backup file on Sunday, incremental backup files on Monday and Tuesday
b) A full backup file on Sunday, an incremental backup file on Tuesday
c) Incremental backup files on Monday and Tuesday
d) An incremental backup file on Tuesday

**thought process:**
sunday - incremental backup (data is after closing time on Monday through Saturday)
wednesday - failure during business hours 
restore data to state of closing time on Tuesday using backup files
~~D cause it says incremental only includes data that has been modified, so this reverts the changes back to before the failure occured~~
~~**Answer: D**~~
needs A because full back up is the base, then Monday and Tuesday contain the changes made in those days
Answer: A

###### Q20. Which of the following is the appropriate description concerning Open Source Software (OSS)?
a) Source code can be edited and then redistributed.
b) Source code is free of charge, but maintenance and support are charged.
c) Copyrights are waived, so it can be used without permission.
d) If the copyright is not waived, the operability must be warranted.
**thought process:**
a prolly, definitely not b, it not C, not D
Answer: A