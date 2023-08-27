# VE280 Final Review (L15-16 & 21-23）

## Lecture 15: Invariants

An invariant is a set of conditions that must always evaluate to true at certain well-defined points; otherwise, the program is incorrect.

### Representation Invariant

For ADT, it is called representation invariant.

It describes the conditions that must hold on those members for the representation to correctly implement the abstraction.  These conditions are not naturally preserved, but need your caution in implementing the class.

#### **Establishing the Invariant**  

Each method in the class can assume that the invariant is true <u>on entry</u> if:

- The representation invariant holds immediately before exiting each <u>method</u>
- Each data element is truly <u>private</u>.  

Therefore, the class is self-consistent.

Essentially, for each method, you should:

-  Do the work of the method (implement)
-  <u>Repair</u> the invariants you broke  (if any)

#### **Checker Function **

<u>defensive programming</u>: write a <u>private</u> method to check whether all invariants are true (before exiting, or after entering, each method):  

```c++
bool repOK();
// EFFECTS: returns true if the rep. invariants hold
```

Next, add the following code right before returning from any function that modifies any of the representation: `assert(repOK())`;
you can write the same line at the beginning of every method too, to check whether the assumption the method relies on is true.  

**Exercise 1**

How should `repOK()` be? What invariant should `load(..)` repair?

```c++
class buckets{
	unsigned int buckets[MAX_SIZE];//buckets initially empty to be loaded
    unsigned int firstLoadedBucket;//the index of first loaded bucket. zero if all empty
    unsigned int bucketLoadedNum;//the number of loaded bucket
    unsigned int loadNum;//the amount of load
    double loadFactor;//(loadNum/MAX_SIZE)
    bool repOK();// EFFECTS: returns true if the rep. invariants hold
    ...
  public:
    void load(unsigned int bucket_index, unsigned int load);
    ...
```

**Solution:**

```c++
bool buckets::repOK(){
    unsigned int true_firstLoadedBucket, true_bucketLoadedNum=0, true_loadNum=0;
    for(unsigned int i=0; i<MAX_SIZE; i++){
        if (buckets[i]!=0){
            true_bucketLoadedNum++;
            true_loadNum+=buckets[i];
        }
    }
    for(int i=0; i<MAX_SIZE; i++){
        if (buckets[i]!=0){
            true_firstLoadedBucket=(unsigned int)i;
            break;
        }
    }
    return true_firstLoadedBucket==firstLoadedBucket && 	
        true_bucketLoadedNum=bucketLoadedNum && true_loadNum=loadNum &&
        loadFactor==((double)true_loadNum)/((double)MAX_SIZE);
}
```

```c++
void buckets::load(unsigned int bucket_index, unsigned int load){
    //assert(repOK());
    if (load!=0){
    	if(buckets[bucket_index]==0)
    		bucketLoadedNum++;
    	buckets[bucket_index]+=load;
    	if(bucket_index<firstLoadedBucket) 
        	firstLoadedBucket=bucket_index;
    	loadNum+=load;
    	loadFactor=((double)loadNum)/((double)MAX_SIZE);
	}
    //assert(repOK());
}
```



## Lecture 16: Dynamic Memory Allocation; Overloading, Default Arguments; Destructor  

### Dynamic Memory Allocation

#### **Types of Variables:**

There are basically three types of variables in C++ programs:

- Global Variables (static)
  - Defined outside functions
  - Lifetime: Their space is set aside before the programs runs (at compile time), and is reserved until the program terminates.
- Local Variables (static)
  - Defined within a block (basically insides curly brackets {})
  - Lifetime: Space is set aside when relevent block is entered (at run time), and reserved until the block is exited.
- Dynamic Variables
  - the compiler doesn't need to know how big it is and how long it lives.  
  - Space is allocated using new, and deallocated using delete by programmer. 

#### **Memory Structure:**

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221113111153410.png" alt="image-20221113111153410" style="zoom:33%;" />

Text stores the code, heap stores dynamic variables and stack stores memory for functions (including main function and local variables).

When stack and heap use up the big void, the program overflows. How many memory they use is indicated by a pointer pointing to the top of heap and another pointing to the bottom of stack.

The "Stack" and "Heap" here refers to specific locations in the memory. They are not equivalent to the data structures called "stack" and "heap", though their behavior is similar.  

#### Dynamic Memory

**Why dynamic memory?**

- Local variables can only be fixed size, we want to change the size at runtime.

```c++
int length;
cin >> length;
int arr[length];
```

This code may be compiled successfully with some compilers. However, it is not allowed in ISO c++ standard. Some compilers allow this as an extension. You can add -pedantic flag to turn on the warning.

- We don't know the exact lifetime of the objects.  

**Allocation**

```c++
Type* obj0 = new Type; // Default construction
Type* obj1 = new Type(); // Default construction
Type* obj2 = new Type(arg1, arg2); // Constructor with 2 params
Type* objA0 = new Type[size]; // Default cons each elt
Type* objA1 = new Type[size](); // Same as obj A0

int* a=new int;//see 'int' as a class 
int* b=new int(1);
int* c=new int[5];
int* d=new int[5]{0};//[0,0,0,0,0]
```

However, you cannot allocate an array of objects using non-default constructors.
When using `new` and `new[]`, following things happens:

1. Allocates space in heap (for one or an array of objects).
2. Constructs object in-place (mainly be calling constructor).
3. Returns the “first” address.

You may remeber `malloc` and `free` in the C language. They only deal with the memory while do nothing to the content in it. That means in C you need to malloc the space and initialize it by hand.
**Deallocation**
Use `delete` and `delete[]` to deallocate single object and arrays respectively. 

**Memory Leak**
The usage of new/delete is very easy. The difficult point is when and where to use them.
Basically, memory leak happens when you <u>lose the address of some dynamic memory</u> (then it would be impossible for you to delete that memory).

**Example:**

```c++
// Each time foo() is called, there is new memory allocated.
// And since p is a local variable, each time p will point to a new address
void foo() {int* p = new int(0); /* Code */}
```

Each time `foo()` is called, some memory is occupied and will not be released even after the program terminates. Gradually, you will drain out all memory of your computer, which is very bad.

#### Check Memory Leak

1. **valgrind**
   Command: `valgrind --leak-check=full ./program <args>`
   Function: search for memory leaks and give details of each individual leak.
   To install, type the command: `sudo apt-get install valgrind ` 

2. **fsanitize**

   compile the program using `g++ -o program <file> -fsanitize=address` and run.

   

### Overloaded Constructor and Default Argument

#### Overloaded Constructor

Functions with same name can have different versions, and compiler tells which function to call based on the actual argument count and types  

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221113153310576.png" alt="image-20221113153310576" style="zoom: 33%;" />

For constructor, programmer can choose default initialization (eg. MAXELTS) or sepcialized (eg. give a size).

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221113153451975.png" alt="image-20221113153451975" style="zoom:33%;" />

#### Default Argument

There's a easier method, using default argument.

Default argument means when you put in no argument, the function can assume a default argument

for example, `IntSet(int size = MAXELTS); `  will realize the function of overloaded constructor with just one function. 

```
IntSet(); //size=MAXELT
IntSet(4); //size=4
```

Mind that you should put default argument to the end

```c++
int add(int a, int b = 0, int c = 1); // OK
int add(int a, int b = 1, int c); // Error
```

Also mind that you don't need to write default argument when implementing.

### Destructor  

**Class's member is dynamic**

When all the members of a class are static, there's no need to write the destructor, since there's a default one that help the compiler automatically released memory of the class's members when the program ends.

However when there are some dynamic members in a class, the compiler can not deal with the heap, so we need to write a destructor to help released them at once.

**Class is dynamic**

The destructor can be called automatically when a class's life ends (like main function or other function ends). But if you dynamically declare a class, it of course should be deleted by programmer, since a dynamic variable can't be deleted by compiler and its destructor won't be called.

**Exercise 2**

How to implement the constructors and the destructor?

```c++
class buckets{
	unsigned int *buckets;//dynamic buckets initially empty to be loaded
    unsigned int firstLoadedBucket;//the index of first loaded bucket
    unsigned int bucketLoadedNum;//the number of loaded bucket
    unsigned int loadNum;//the amount of load
    double loadFactor;//(loadNum/MAX_SIZE)
    double maxLoadFactor;//upper bound of loadFactor
  public:
    buckets(unsigned int size=MAX_SIZE, double max_factor=2);
    buckets(unsigned int load; unsigned int index;
            unsigned int size=MAX_SIZE, double max_factor=2);
    ~buckets();
    ...
```

**Solution:**

```c++
buckets::buckets(size, max_factor):buckets(new unsigned int[size]{0}), firstLoadedBucket(0), bucketLoadedNum(0), loadNum(0), loadFactor(0), maxLoadFactor(max_factor){}
buckets::buckets(load, index, size, max_factor):buckets(new unsigned int[size]{0}), loadNum(load), loadFactor((double)load/(double)size), maxLoadFactor(max_factor){
    if (load==0){
        firstLoadedBucket=0; 
        bucketLoadedNum=0;
    }else{
        firstLoadedBucket=index; 
        bucketLoadedNum=1;
        buckets[index]=load;
    }
}
buckets::~buckets(){
    delete[] buckets;
}
```



## Lecture 21: Operator Overloading

C++ lets us redefine the meaning of the operators when applied to objects of class type. This is known as operator overloading. Like any other function, an overloaded operator has a return type and a parameter list.

### Unary operators

An overloaded unary operator has no (explicit) parameter if it is a member function and one parameter if it is a nonmember function.

Examples:

```c++
//you don't have to remember these
A& A::operator++(); // ++a
A A::operator++(int); // a++
A& A::operator--(); // --a
A A::operator--(int); // a--
void operator++(& A); // ++a for nonmember function
```

### Binary operators

An overloaded binary operator would have one parameter when defined as a member and two parameters when defined as a nonmember function.

Examples:

```c++
//you should remember the argument and return types
A& A::operator= (const A& rhs);
A& A::operator+= (const A& rhs);
A& A::operator-= (const A& rhs);
A operator+ (const A& lhs, const A& rhs);//similar for -, *, / ...
A A::operator+(const A &r);//returns *this “+” r
istream& operator>> (istream& is, A& rhs);
ostream& operator<< (ostream& os, const A& rhs);
bool operator== (const A& lhs, const A& rhs);
bool operator!= (const A& lhs, const A& rhs);
VE280 2021FA Final RC part1.md 2021/12/10
6 / 20
bool operator< (const A& lhs, const A& rhs);
bool operator<= (const A& lhs, const A& rhs);
bool operator> (const A& lhs, const A& rhs);
bool operator>= (const A& lhs, const A& rhs);
// especially
const T& A::operator[] (size_t pos) const;
T& A::operator[] (size_t pos);  
```

### Friendship


We may want to access private member of class instances. You could provide an accessing operator for each of the member, but often it is not a good idea. One workaround is specifically grant access to the protected members. This can be done by using the friend keyword:
```c++
class Bar {
friend void foo (const MyClass &mc);
}
```

It doesn’t matter where this is marked public or private.
friend can also grant access to classes, and Baz now can access private member of Bar:

```c++
class Bar {
friend class Baz;
}
```


Pay attention that friend is not mutual. If Class A declares Class B as friend. Class B can access Class A’s private member, but the other way around doesn’t work.  

Make binary operators friend functions:

```c++
class Complex {
// OVERVIEW: a complex number class
double real;
double imag;
public:
Complex(double r=0, double i=0);
Complex &operator += (const Complex &o);
friend Complex operator+(const Complex &o1, const Complex &o2);
};
```



## Lecture 22: Linear List; Stack  

### Linear List

A collection of zero or more integers; duplicates possible. It supports insertion and removal by position.

#### Insertion

```c++
void insert(int i, int v) // if 0 <= i <= N (N is the size of the list),
//insert v at position i; otherwise, throws BoundsError exception.
```

Example: L1 = (1, 2, 3)
L1.insert(0, 5) = (5, 1, 2, 3);
L1.insert(1, 4) = (1, 4, 2, 3);
L1.insert(3, 6) = (1, 2, 3, 6);
L1.insert(4, 0) throws BoundsError

#### Removal

```c++
void remove(int i) // if 0 <= i < N (N is the size of the list), remove the i-th
//element; otherwise, throws BoundsError exception.
```

Example: L2 = (1, 2, 3)
L2.remove(0) = (2, 3);
L2.remove(1) = (1, 3);
L2.remove(2) = (1, 2);
L2.remove(3) throws BoundsError

### Stack

Property: **last in, first out (LIFO)**

- ``size()`` : number of elements in the stack.
- ``isEmpty()`` : check if stack has no elements.
- ``push(Object o) ``: add object o to the top of the stack.
- ``pop() ``: remove the top object if stack is not empty; otherwise, throw stackEmpty .
- ``Object &top()`` : returns a reference to the top element.


#### Implementation Comparison

1. Using Arrays
   Need an int size to record the size of the stack.

- ``size()`` : return size;
- ``isEmpty()`` : return (size == 0);
- ``push(Object o) ``: add object o to the top of the stack and increment size . Allocate more space if necessary.
- ``pop() ``: If isEmpty() , throw stackEmpty ; otherwise, decrement size .
  Object &top() : returns a reference to the top element Array[size-1].

2. Using Linked Lists

- ``size()`` : LinkedList::size();
- ``isEmpty()`` : LinkedList::isEmpty();
- ``push(Object o)`` : insert object at the beginning LinkedList::insertFirst(Object o); .
- ``pop()`` : remove the first node LinkedList::removeFirst();
  Object &top() : returns a reference to the object stored in the first node

## Lecture 23: Queue

- ``size()``: number of elements in the queue.
- ``isEmpty()``: check if queue has no elements.
- ``enqueue(Object o)``: add object o to the rear of the queue.
- ``dequeue()``: remove the front object of the queue if not empty; otherwise, throw
  queueEmpty.
- ``Object &front()``: return a reference to the front element of the queue.
- ``Object &rear()``: return a reference to the rear element of the queue.


#### Queues Using Linked Lists


How to implement the mothods using linked lists:

- `` enqueue(Object o)``: add a node to the end of the linked list.

- `` dequeue()``: remove a node from the head of the linked list.

- `` size()``: can iterate through the linked list and count the number of nodes.

- `` isEmpty()``: check if the pointer to the linked list is NULL.

- `` Object &front()``: returns a reference to the node at the head of the linked list.

- `` Object &rear()``: returns a reference to the node at the end of the linked list.

 #### Queues Using Arrays

 Let the elements "drift" within the array.

 Maintain two integers to indicate the front and the rear of the queue (advance front when dequeuing; advance rear when inserting).

 Use a circular array (more space efficient):

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221201160731878.png" alt="image-20221201160731878" style="zoom: 25%;" />

 When inserting a new element, advance rear circurlarly; when popping out an element,

 advance front circularly.

 Can be realized by

```c++
front = (front+ 1 ) % MAXSIZE;
rear = (rear+ 1 ) % MAXSIZE;
```

 Solve the problem of distinguishing an empty queue and full queue:

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221213160959211.png" alt="image-20221213160959211" style="zoom:25%;" />

 Maintain a ``flag`` indicating empty or full, or a ``count`` on the number of elements in the queue.

 We can see that using array can be more complicated than linked list, and the size can be restricted by ``MAXSIZE`` of array. However, it can be easier to access the elements by index. For example, the index of the second element can be ``front+1 ``(if not exceeding the capacity of array). Still, accessing elements by index is not very useful in queues.

### Deque

 - Not a proper English word, pronounced as “deck”.

 - Means double-ended queue

- Property: Items can be inserted and removed from both ends of the list.

- Methods:
  - ``push_front(Object o)``
  - ``push_back(Object o)``
  - ``pop_front()``
  - ``pop_back()``

The implementation can be more complicated than queue. Only use it if **inserting and removing from both ends are truly necessary**.
