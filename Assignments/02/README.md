# CS370
## Assignment 02

### Part 1:

#### Implementation

The implemtentation for this section was pretty simple. Following 7.4.2.6 from the book it becamse very obvious that the way to implement this is to simply add `n->quanta = p->quanta;` after the null check of the parent process.

#### Testing

I actually mixed the testing of this and part 2 together. I will explain that in the section for Part 2.


### Part 2:

#### Implementation

I was initially confused what we need to do here, we hadn't really covered what `devpro` did in class or how to use it. I read a few man pages and realized that it is simply a "debugger" of sorts.

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

I had to then add the description of this method into `interp.h`.

#### Testing

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
			iquanta(r);
```

#### Testing

So to test this I just simply made vmachine print out the quanta for each process that was being ran. If I saw anything increase from the amount of 2048 (PQUANTA), then I knew it ran successfully.

```
			print("The quanta for the current process is: %d\n",r->quanta);
```

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

#### Testing

I, again, used printing to test that these were being added correctly. It was hard to notice with printing so I decided to try it with something else.

I remember that in class Prof. Stuart told us that in the `bounce` game, each ball is a process on it's own so I knew if this change actually worked that game shouldn't be able to load. Once I tested it, it did not load and basically locked the entire system.
