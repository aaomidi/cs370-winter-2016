# CS370
## Assignment 02

### Part 1:

#### Implementation

The implemtentation for this section was pretty simple. Following 7.4.2.6 from the book it becamse very obvious that the way to implement this is to simply add `n->quanta = p->quanta;` after the null check of the parent process.

#### Testing

I actually mixed the testing of this and part 2 together. I will explain that in the section for Part 2.

Another way I implemented the testing was editing `devprog.c` and adding the quanta to `progread`'s `Qstatus`. This way I can check the quanta for all of the processes.

#### Implications

Honestly this changed nothing in the OS. The quanta has no chance to change so inheriting or not inherting does absolutely nothing. 

### Part 2:

#### Implementation

I was initially confused what we need to do here, we hadn't really covered what `devprog` did in class or how to use it. I read a few man pages and realized that it is simply a communication line into the programs. You could say it acted as a "debugger" for the processes.

What I did to implement this is add the following code to the bottom of the file:

```
int
progquanta(Prog* p, int nq) // Places an upper bound of 10000 and lower bound of 100
{
	if(nq < 100||nq > 10000)
		return -1;

	p->quanta = nq;

	return 0;
}
```

I had to then add the description of this method into `interp.h`. (I'll explain why in testing).

#### Testing and Implications

So the reason I added it to `interp.h` was to be able to test it through newprog. What i did to test is:
```
	Prog* head = isched.head;
	Prog* tail = isched.tail;
	while(head->next != nil){
		if(head->pid==n->pid) { // We actually don't need this since it isn't scheduled yet.
			print("ignoring the current process\n");
		}
		progquanta(head, 1); // This is after I remove the bound checking
		head = head->next;
	}
```

This makes all the processes other than the one we just made get a tiny quanta. Making them VERY slow in doing what they're supposed to do. 

The last process still preserves it's PQUANTA slice. 

Basically this makes it so if I run test1 in one shell inside `wm/wm` and then make another shell and run it again, the second `test1` will catch up and pass the first one in a matter of seconds.

This way I can make sure both my first part and second part of this assignment is correct :)

Another way I tested this after I met with Prof. Stuart was to actually add a system to control the process. I did this by editing `devprog.c`. The code I added is as follows:

```
		case CMquanta:
			quanta = atoi(cb->f[1]);
			print("Changing the quanta of process %d to %d\n", p->pid, quanta);
			progquanta(p, quanta);
			break;
```

The debug message and the checking of status explained in part 1 allows me to make sure this works.

### Part 3

#### Implementation

This one was just as easy, I used the code i developed in part 2:
```
int 
iquanta(Prog *p)
{
	int q = p->quanta;

	q += q / 2;

	return progquanta(p, q);	
}
```

and the following code inside vmachine()

```
			FPrestore(&o->fpu);
			r->xec(r);
			FPsave(&o->fpu);

			// Change the quanta
			if(isched.runhd != nil) { // If there is something to schedule
				//print("X\n");
				if(r == isched.runhd) { // The one that I'm running has to be the head of the list.
					//print("Y\n");
					if(isched.runhd != isched.runtl) { // If there is more than one thing in the list
						isched.runhd = r->link;
						r->link = nil;
						isched.runtl->link = r;
						isched.runtl = r;
						
					}
					iquanta(r); // The process wasn't blocked.
				}
			}
```

#### Testing

So to test this I just simply made vmachine print out the quanta for each process that was being ran. If I saw anything increase from the amount of 2048 (PQUANTA), then I knew it ran successfully.

```
			print("The quanta for the current process is: %d\n",r->quanta);
```

Another way to test it is just to look into the status file, see if the quanta changes. (Spoiler Alert: It does)

#### Implications

An implication of this new quanta system will be that if a program actually needs the extra processing power, it's going to get it (up to a certain limit). This is good for servers, it ensures that the processing power is where it should be at and not wandering around the system.

### Part 4

#### Implementation

This was challenging at first since I had to figure out exactly the structure of the linked list and what each linked structure pointed to. I went back and watched the lectures and they made a lot more sense in context.

This is how I implemented this:

```
	p->state = Pready;
	p->link = nil;

	if(isched.runhd == nil) { // If no element
		isched.runhd = p;
		isched.runtl = p;
	} else if (isched.runhd == isched.runtl) { // If only one element
		isched.runhd->link = p;
		isched.runtl = p;
	} else { // If more than one element
		Prog *s = isched.runhd;
		Prog *l = s->link;
		s->link = p;
		p->link = l;
	}
```

I replaced the code under the if statement with this.

#### Testing and Implications

I, again, used printing to test that these were being added correctly. It was hard to notice with printing so I decided to try it with something else.

I remember that in class Prof. Stuart told us that in the `bounce` game, each ball is a process on it's own so I knew if this change actually worked that game shouldn't be able to load. Once I tested it, it did not load and basically locked the entire system.

Although I thought this was a natural symptom, apparently it isn't? I know what is causing it, but I don't actually know how to solve it. When a process gets blocked, it seems to put itself "next in line" and doesn't go to the back of the ready progs. This way if there are two blocked processes, it just goes on an endless loop of trying to unlock itself.

What I did to make sure this doesn't block the system I did this instead:

```
void
addrun(Prog *p) // Transition from blocked to ready
{
	if(p->addrun != 0) { // Debug state stuff, ignore
		p->addrun(p);
		return;
	}

	p->link = nil;

	if(p->state==Palt || p->state==Psend || p->state==Precv){ // If blocked
		p->state = Pready;
		p->link = nil;
		if(isched.runhd == nil)
			isched.runhd = p;
		else
			isched.runtl->link = p;

		isched.runtl = p;
	} else {
		print("Called\n");
		p->state = Pready;

		if(isched.runhd == nil) { // If no element
			isched.runhd = p;
			isched.runtl = p;
		} else if (isched.runhd == isched.runtl) { // If only one element
			isched.runhd->link = p;
			isched.runtl = p;
		} else { // If more than one element
			//print("PID of Head: %d\n\tPID of 2nd: %d\n\tPID of next to add: %d\n",isched.runhd->pid,isched.runhd->link->pid,p->pid);
			Prog *s = isched.runhd;
			Prog *l = s->link;
			p->link = l;
			s->link = p;
			//print("\tAFTER CHANGE PID of Head: %d\n\tPID of 2nd: %d\n\tPID of next to add: %d\n",isched.runhd->pid,isched.runhd->link->pid,isched.runhd->link->link->pid);
		}
	}
}
```

This is a hacky solution but it works.

One last thing I noticed. About the maximum and minimum quanta:
- If the quanta is too small the overhead of context switching becomes larger than the time the program can run. In my testing I found out I shouldn't go below ~100.
- If the quanta is too big, you effectively have a batch system with just no priority and jobs going in as FIFO. The maximum quanta however depends in this case on the hardware of the system its running on. On my machine I really didnt notice anything until around ~30000.
