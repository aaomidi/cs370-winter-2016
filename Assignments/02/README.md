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
void
progquanta(Prog* p, int nq)
{
	p->quanta = nq;
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
		progquanta(head,1);
		head = head->next;
	}
```

This makes all the processes other than the one we just made get a tiny quanta. Making them VERY slow in doing what they're supposed to do. 

The last process still preserves it's PQUANTA slice. 

Basically this makes it so if I run test1 in one shell inside `wm/wm` and then make another shell and run it again, the second `test1` will catch up and pass the first one in a matter of seconds.

This way I can make sure both my first part and second part of this assignment is correct :)

### Part 3