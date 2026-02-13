# ooSimple-Transpiler

The **ooSimple-Transpiler** is a source-to-source compiler that translates a custom object-oriented programming language (`ooSimple`) into equivalent C code.

The project is implemented using the ANTLR4 framework and demonstrates a complete language-processing pipeline, including parsing, semantic analysis with scoped symbol tables, and structured C code generation.

Unlike a traditional compiler that targets machine code, this transpiler focuses on transforming high-level object-oriented abstractions into low-level procedural constructs while preserving program semantics. The goal is to make object-oriented behavior explicit within a pure C environment.

Key highlights of the project include:

- **Custom Object-Oriented Language Design**  
  The `ooSimple` language supports classes, constructors, methods, multiple inheritance, typed variables, control-flow constructs, and structured program entry via a `Main` class.

- **ANTLR4-Based Front-End**  
  Grammar-driven lexical and syntactic analysis using ANTLR4, producing a parse tree that serves as the foundation for semantic processing.

- **Two-Phase Semantic Processing**  
  The compiler performs structured tree traversals to construct symbol tables, enforce scope rules, validate inheritance relationships, and perform semantic checks before code generation.

- **Object-Oriented to C Mapping Strategy**  
  Classes are translated into C `struct` types.  
  Methods become standalone C functions receiving an explicit `self` pointer.  
  Constructors allocate and initialize heap-allocated structures.  
  Multiple inheritance is implemented using explicit parent pointers (`super_*` fields), preserving access to inherited state.

- **Static Method Binding**  
  Method overriding is resolved during transpilation.  
  The generated C code does not rely on virtual tables or runtime dispatch; instead, method calls are statically bound during code generation, keeping the output predictable and transparent.

- **Readable and Deterministic C Output**  
  The produced C programs are human-readable, modular, and suitable for inspection, debugging, or further compilation using a standard C compiler.

This project serves both as a practical demonstration of compiler construction principles and as a study in how object-oriented abstractions can be systematically lowered into procedural code.



## ANTLR4 Overview

ANTLR4 (Another Tool for Language Recognition) is a parser generator used to build compilers, interpreters, and translators for custom languages. In this project, it is used to define and process the `ooSimple` language. Key aspects include:

### Stages of ANTLR4 Processing

1. **Grammar Definition**: The user defines the grammar of the target language in a `.g4` file. This grammar specifies the rules for tokenizing and parsing the input.
    - Example: `ooSimpleGrammar.g4` describes the structure of the `ooSimple` language.
    - Grammar rules are categorized into lexer rules (for tokens) and parser rules (for syntax structure).

2. **Lexer Generation**: ANTLR4 generates a lexer from the grammar file. The lexer scans the input code and breaks it into tokens based on the rules defined in the grammar.
    - Tokens are the smallest meaningful units like keywords (`class`), identifiers (`myVar`), or symbols (`{`, `}`).
    - The lexer uses regular expressions defined in the grammar to recognize these tokens.

3. **Parser Generation**: ANTLR4 generates a parser that organizes the tokens into a structured hierarchy according to the grammar rules.
    - The parser checks the syntax of the token stream, ensuring it follows the defined grammar.
    - Errors in syntax are reported during this stage.
    - The output of this step is the **parse tree**, which represents the logical structure of the input.

4. **Parse Tree Construction**: The parse tree is a hierarchical representation of the input, where each node corresponds to a rule or token.
    - **Root Node**: Represents the starting rule defined in the grammar.
    - **Child Nodes**: Represent the components or subrules of the parent node.
    - **Leaf Nodes**: Represent individual tokens (e.g., keywords or literals).

5. **Listener or Visitor Execution**: Custom logic can be implemented by traversing the parse tree using Listeners or Visitors to perform tasks like semantic analysis or code generation.
    - Visitors provide a more manual approach, offering greater control over tree traversal.

### Parse Tree and Its Creation

The parse tree is central to how ANTLR4 processes input. Here's how it works:

- **Creation**: ANTLR4 builds the parse tree as the parser processes the input code. Each node in the tree corresponds to a specific grammar rule or token.
- **Structure**: The root node represents the starting grammar rule, and child nodes represent the components of that rule. Leaf nodes are individual tokens.
- **Traversal**: The parse tree can be traversed automatically using Listeners or manually using Visitors.
- **Purpose**: The parse tree serves as a blueprint for further processing, like validating semantics or generating code.

### What Are Listeners?

Listeners are a mechanism provided by ANTLR4 to traverse the parse tree. They automatically respond to events triggered during tree traversal, such as entering or exiting a node.

- **Built-In Methods**:
    - `enterRule`: Called when the parser enters a specific grammar rule.
    - `exitRule`: Called when the parser exits that rule.
- **Custom Implementation**:
    - Users extend the base listener class to add logic for handling specific nodes in the tree.
    - Example: In this project, the `SymbolicTablePhase` listener constructs and validates the symbol table.

### Relationship Between Listeners and the Parse Tree

Listeners interact directly with the parse tree to process the input. Here's how:

1. **Invocation**: As the parse tree is traversed, ANTLR4 automatically invokes the appropriate listener methods (e.g., `enterRule` and `exitRule`) for each node.
2. **Custom Logic Execution**: Listeners implement custom actions for each relevant node, such as:
    - Building a symbol table (e.g., `SymbolicTablePhase` in this project).
    - Generating target code (e.g., `CodeGenerator`).
3. **Event-Driven Processing**: The listener pattern allows event-driven processing, enabling modular and efficient handling of grammar rules.
4. **Efficiency**: Listeners reduce the complexity of manual tree traversal by focusing on specific nodes, allowing modular and organized logic.
5. **Integration**: Each listener method corresponds to a specific rule in the grammar, ensuring a tight connection between the grammar's design and the tree's processing.

By combining the parse tree's hierarchical structure with listeners' event-driven approach, ANTLR4 provides a powerful and flexible framework for language processing.

## Language Design

The `ooSimple` programming language is a custom object-oriented language designed with the following key features:

### Class Definitions

- Classes are declared using the keyword `class`, followed by the class name.
- **Multiple Inheritance**: Supported using the keyword `inherits`. A class may inherit from one or more parent classes, allowing complex hierarchies.
- Example:
  ```python
  class MyClass inherits ParentClass1, ParentClass2:
      ...
  ```

### Method Definitions

- Methods are defined using the keyword `def`, followed by the method name and parameters.
- Methods can return values or modify the object's state.
- The `self` keyword is used as the first parameter to refer to the current instance.
- The return type is specified after a colon `:`. If the method does not return a value, a dash (`-`) is used to indicate `void`.
- Example:
  ```python
  def calculate(self, value): int
      return value * 2;
  ```

### Constructor Definitions

- Constructors are special methods named `__init__` and are used to initialize object properties.
- Example:
  ```python
  def __init__ClassName(self, name): int
      self.name = name;
      ;
  ```

### Main Class

- A special `main` class can be defined to contain the entry point of the program.
- Example:
  ```python
  class Main:
      def main(self):
          print "Hello, World!";
          ;
  ```

### Variables and Declarations

- Variables are declared with their types, such as `int` or `class` types.
- Multiple variables can be declared in a single line.
- Example:
  ```python
  int a, b, c;
  int x;
  int z;
  ;
  ```

### Control Flow Statements

- Standard control flow constructs are supported, such as `if-else`, `while`, and `return`.
- Example of `if-else`:
  ```python
  if (x > 0):
      print "Positive";
  else:
      print "Non-positive";
  endif;
  ```
- Example of `while`:
  ```python
  while (x > 0):
      x = x - 1;
      print x;
  endwhile;
  ```
The ooSimpleGrammar.g4 file describe fully the language structure.

## Compiler Process Flow

The **ooSimple-Transpiler** compiler processes the input `ooSimple` code in a systematic and multi-step approach to produce equivalent C code. The process includes the following stages:

1. **Parse Tree Generation**:
    - The compiler begins by parsing the `ooSimple` input code using the ANTLR4 framework.
    - A hierarchical parse tree is created based on the grammar rules defined in `ooSimpleGrammar.g4`.
    - This tree represents the syntactic structure of the input program, with nodes corresponding to grammar rules or tokens.

2. **First Tree Traversal: Symbol Table Creation**:
    - During the first traversal of the parse tree, a symbol table is constructed.
    - This table stores metadata for program entities, including:
        - **Classes**: Their names, inheritance relationships, and members.
        - **Methods**: Their names, parameter types, return types, and visibility.
        - **Variables**: Their names, types, and scopes.
    - Scoping rules are enforced to manage nested blocks and inheritance.
    - Semantic checks, such as duplicate declarations or undefined parent classes, are performed.

3. **Second Tree Traversal: Code Generation**:
    - The parse tree is traversed a second time to produce equivalent C code.
    - The traversal involves:
        - **Statement Translation**: High-level constructs like `if`, `while`, and `return` are converted to C equivalents.
        - **Variable Resolution**: Ensuring all variables and methods are declared in the symbol table before use.
        - **Output Construction**: Sequentially generating lines of valid C code to replicate the behavior of the input program.
    - Type consistency and scoping rules are revalidated to ensure correctness.

4. **Error Handling**:
    - Syntax errors are reported during the parsing stage.
    - Semantic errors, such as undeclared variables or type mismatches, are reported during the first and second traversals.


---

# How to Run the Project

## Requirements

- Java 21 installed
- Java available in system PATH
- Bash environment (Linux / macOS / WSL / Git Bash on Windows)


## Running the Transpiler

The project is executed through the `run_antlr.sh` script located in the root directory.

```bash
./run_antlr.sh
```
## Input and Output Configuration

The input file is defined in the final command of the `run_antlr.sh` script, where the `.oos` source file is passed as an argument to the `antrl4.Compiler` class. To transpile a different program, simply replace the file path inside the quotation marks with the desired `.oos` file.

The generated C programs are written to the `compiler_output` directory.

---

## Example Program and Generated Output

Below is a sample `ooSimple` program and the corresponding C code produced by the transpiler.

### Input (`Vehicle.oos`)

```text
class Vehicle:
    int capacity;
    int year;
    ;

    def __init__(self): Vehicle
        int temp;;
        temp = 0;
        self.capacity = temp;
        self.year = 0;
        ;

    def __init__(self, int capacity, int year): Vehicle
        self.capacity = capacity;
        self.year = year;
        ;

    def setCapacity(self, int capacity): -
        self.capacity = capacity;
        ;

    def setYear(self, int x): -
        self.year = x;
        ;

    def getCapacity(self): int
        return self.capacity;
        ;

    def getYear(self): int
        return self.year;
        ;


    def getFieldsSum(self): int
        return self.capacity + self.year;
        ;


    def equals(self, Vehicle other): int
        if([self.capacity == other.capacity] and [self.year == other.year]):
            return 1;
        else:
            return 0;
        endif;
        ;




class Car inherits Vehicle:
    int doorsNumber;;

    def __init__(self, int capacity , int year, int doorsNumber): Car
        self.capacity = capacity;
        self.year = year;
        self.doorsNumber = doorsNumber;
        ;

    def __init__(self,int doorsNumber): Car
        self.capacity = 0;
        self.year = 0;
        self.doorsNumber = doorsNumber;
            ;


    def setDoorsNumber(self, int x): -
        self.doorsNumber = x;
        ;

    def getDoorsNumber(self): int
        return self.doorsNumber;
        ;

    def getYearAsCar(self): int
        return self.year;
        ;

    def setYearAsCar(self, int year): -
        self.year = year;
        ;


    def getFieldsSum(self): int
        return self.capacity + self.year + self.doorsNumber;
        ;




class Buss inherits Vehicle,Car:
    int ticketPrice;
    ;

    def __init__(self,int capacity , int year , int doorsNumber, int ticketPrice): Buss
        self.capacity = capacity;
        self.year = year;
        self.doorsNumber = doorsNumber;
        self.ticketPrice = ticketPrice;
        ;


        def __init__(self,int capacity , int year , int doorsNumber): Buss
                self.capacity = capacity;
                self.year = year;
                self.doorsNumber = doorsNumber;
                ;

        def __init__(self,int capacity , int ticketPrice): Buss
            self.ticketPrice = 0;
            self.ticketPrice = ticketPrice;
            self.capacity = 0;
            self.doorsNumber = 1 - 1;
            ;


        def getFieldsSum(self): int
            return self.capacity + self.year + self.doorsNumber + self.ticketPrice;
            ;

        def setTicketPrice(self, int newPrice): -
            self.ticketPrice = newPrice;
            ;


        def getTicketPrice(self): int
            return self.ticketPrice;
            ;







####################################


class Main:
    def main(self): -
        Vehicle v0, v1;
        Car c1;
        Buss b1;
        int x;
        ;

        v0 = Vehicle();
        print "v0 Capacity : ";
        print v0.getCapacity();
        print "v0 Year : ";
        print v0.getYear();

        v1 = Vehicle(7,1997);
        print "v1 Capacity : ";
        print v1.getCapacity();
        print "v1 Year : ";
        print v1.getYear();

        v1.setCapacity(10);
        v1.setYear(2003);
        print "v1 Capacity : ";
        print v1.getCapacity();
        print "v1 Year : ";
        print v1.getYear();

        c1 = Car(2,2018,3);
        print "Get year as vehicle : ";
        print c1.getYear();
        print "Get year as car : ";
        print c1.getYearAsCar();

        c1.setYear(1945);
        c1.setYearAsCar(2000);

        print "Get fields Sum. Should override vehicle : ";
        print c1.getFieldsSum();


        b1 = Buss(80,2024,3,5);
        print "Buss capacity :  year:   doorsNumber:    ticketPrice:    \n";
        print  b1.getCapacity(), b1.getYear(), b1.getDoorsNumber(), b1.getTicketPrice();


       print b1.getFieldsSum();

       b1.setDoorsNumber(1);
       print b1.getDoorsNumber();

       print b1.equals(v1);

       x = b1.getCapacity() + b1.getYear() + b1.getDoorsNumber() + b1.getTicketPrice();
       ;
```

## Generated C Output (`Vehicle.c`)

This example demonstrates the transformation of an object-oriented `ooSimple` program (`Vehicle.oos`) into equivalent C code (`Vehicle.c`).

Classes are translated into C `struct` types, methods become standalone C functions receiving an explicit `self` pointer, and inheritance is implemented through explicit parent pointers (`super_*` fields).

Inheritance is implemented using explicit parent pointers (`super_*` fields).
Each derived structure dynamically allocates and maintains pointers to its parent structures, enabling access to inherited fields while preserving the hierarchical relationships defined in the original object-oriented design.

The transpiler supports multiple inheritance by maintaining separate parent pointers for each inherited class.

```text
#include <stdio.h>
#include <stdlib.h>

//Structs declaration
typedef struct Vehicle {
	int year;
	int capacity;
} Vehicle;

typedef struct Car {
	Vehicle* super_1;
	int doorsNumber;
} Car;

typedef struct Buss {
	Vehicle* super_1;
	Car* super_2;
	int ticketPrice;
} Buss;


//Methods Declarations
Vehicle* Vehicle_Constructor();
Vehicle* Vehicle_Constructor_int_int(int, int);
void setCapacity_Vehicle_int(Vehicle*, int);
void setYear_Vehicle_int(Vehicle*, int);
int getCapacity_Vehicle(Vehicle*);
int getYear_Vehicle(Vehicle*);
int getFieldsSum_Vehicle(Vehicle*);
int equals_Vehicle_Vehicle(Vehicle*, Vehicle*);
Car* Car_Constructor_int_int_int(int, int, int);
Car* Car_Constructor_int(int);
void setDoorsNumber_Car_int(Car*, int);
int getDoorsNumber_Car(Car*);
int getYearAsCar_Car(Car*);
void setYearAsCar_Car_int(Car*, int);
int getFieldsSum_Car(Car*);
Buss* Buss_Constructor_int_int_int_int(int, int, int, int);
Buss* Buss_Constructor_int_int_int(int, int, int);
Buss* Buss_Constructor_int_int(int, int);
int getFieldsSum_Buss(Buss*);
void setTicketPrice_Buss_int(Buss*, int);
int getTicketPrice_Buss(Buss*);

int main(void){
	int x;
	Vehicle* v0;
	Vehicle* v1;
	Car* c1;
	Buss* b1;
	v0 = Vehicle_Constructor();
	printf("v0 Capacity : ");
	printf("%d""\n", getCapacity_Vehicle(v0));
	printf("v0 Year : ");
	printf("%d""\n", getYear_Vehicle(v0));
	v1 = Vehicle_Constructor_int_int(7, 1997);
	printf("v1 Capacity : ");
	printf("%d""\n", getCapacity_Vehicle(v1));
	printf("v1 Year : ");
	printf("%d""\n", getYear_Vehicle(v1));
	setCapacity_Vehicle_int(v1,10);
	setYear_Vehicle_int(v1,2003);
	printf("v1 Capacity : ");
	printf("%d""\n", getCapacity_Vehicle(v1));
	printf("v1 Year : ");
	printf("%d""\n", getYear_Vehicle(v1));
	c1 = Car_Constructor_int_int_int(2, 2018, 3);
	printf("Get year as vehicle : ");
	printf("%d""\n", getYear_Vehicle(c1->super_1));
	printf("Get year as car : ");
	printf("%d""\n", getYearAsCar_Car(c1));
	setYear_Vehicle_int(c1->super_1,1945);
	setYearAsCar_Car_int(c1,2000);
	printf("Get fields Sum. Should override vehicle : ");
	printf("%d""\n", getFieldsSum_Car(c1));
	b1 = Buss_Constructor_int_int_int_int(80, 2024, 3, 5);
	printf("Buss capacity :  year:   doorsNumber:    ticketPrice:    \n");
	printf("%d"" %d"" %d"" %d""\n", getCapacity_Vehicle(b1->super_1) ,getYear_Vehicle(b1->super_1) ,getDoorsNumber_Car(b1->super_2) ,getTicketPrice_Buss(b1));
	printf("%d""\n", getFieldsSum_Buss(b1));
	setDoorsNumber_Car_int(b1->super_2,1);
	printf("%d""\n", getDoorsNumber_Car(b1->super_2));
	printf("%d""\n", equals_Vehicle_Vehicle(b1->super_1,v1));
	x = getCapacity_Vehicle(b1->super_1) + getYear_Vehicle(b1->super_1) + getDoorsNumber_Car(b1->super_2) + getTicketPrice_Buss(b1);
	return 0;
}

//Constructors Implementations
Vehicle* Vehicle_Constructor(){
	Vehicle* str_ = (Vehicle*) malloc(sizeof(Vehicle));
	int temp;
	temp = 0;
	str_->capacity = temp;
	str_->year = 0;
	return str_;
}
Vehicle* Vehicle_Constructor_int_int(int capacity, int year){
	Vehicle* str_ = (Vehicle*) malloc(sizeof(Vehicle));
	str_->capacity = capacity;
	str_->year = year;
	return str_;
}
Car* Car_Constructor_int_int_int(int capacity, int year, int doorsNumber){
	Car* str_ = (Car*) malloc(sizeof(Car));
	str_->super_1 = (Vehicle*) malloc(sizeof(Vehicle));
	str_->super_1->capacity = capacity;
	str_->super_1->year = year;
	str_->doorsNumber = doorsNumber;
	return str_;
}
Car* Car_Constructor_int(int doorsNumber){
	Car* str_ = (Car*) malloc(sizeof(Car));
	str_->super_1 = (Vehicle*) malloc(sizeof(Vehicle));
	str_->super_1->capacity = 0;
	str_->super_1->year = 0;
	str_->doorsNumber = doorsNumber;
	return str_;
}
Buss* Buss_Constructor_int_int_int_int(int capacity, int year, int doorsNumber, int ticketPrice){
	Buss* str_ = (Buss*) malloc(sizeof(Buss));
	str_->super_1 = (Vehicle*) malloc(sizeof(Vehicle));
	str_->super_2 = (Car*) malloc(sizeof(Car));
	str_->super_1->capacity = capacity;
	str_->super_1->year = year;
	str_->super_2->doorsNumber = doorsNumber;
	str_->ticketPrice = ticketPrice;
	return str_;
}
Buss* Buss_Constructor_int_int_int(int capacity, int year, int doorsNumber){
	Buss* str_ = (Buss*) malloc(sizeof(Buss));
	str_->super_1 = (Vehicle*) malloc(sizeof(Vehicle));
	str_->super_2 = (Car*) malloc(sizeof(Car));
	str_->super_1->capacity = capacity;
	str_->super_1->year = year;
	str_->super_2->doorsNumber = doorsNumber;
	return str_;
}
Buss* Buss_Constructor_int_int(int capacity, int ticketPrice){
	Buss* str_ = (Buss*) malloc(sizeof(Buss));
	str_->super_1 = (Vehicle*) malloc(sizeof(Vehicle));
	str_->super_2 = (Car*) malloc(sizeof(Car));
	str_->ticketPrice = 0;
	str_->ticketPrice = ticketPrice;
	str_->super_1->capacity = 0;
	str_->super_2->doorsNumber = 1 - 1;
	return str_;
}

//Methods Implementations
void setCapacity_Vehicle_int(Vehicle* str_, int capacity){
	str_->capacity = capacity;
}

void setYear_Vehicle_int(Vehicle* str_, int x){
	str_->year = x;
}

int getCapacity_Vehicle(Vehicle* str_){
	return str_->capacity;
}

int getYear_Vehicle(Vehicle* str_){
	return str_->year;
}

int getFieldsSum_Vehicle(Vehicle* str_){
	return str_->capacity + str_->year;
}

int equals_Vehicle_Vehicle(Vehicle* str_, Vehicle* other){
	if((str_->capacity == other->capacity) && (str_->year == other->year)){
		return 1;
	}else {
		return 0;
	};
}

void setDoorsNumber_Car_int(Car* str_, int x){
	str_->doorsNumber = x;
}

int getDoorsNumber_Car(Car* str_){
	return str_->doorsNumber;
}

int getYearAsCar_Car(Car* str_){
	return str_->super_1->year;
}

void setYearAsCar_Car_int(Car* str_, int year){
	str_->super_1->year = year;
}

int getFieldsSum_Car(Car* str_){
	return str_->super_1->capacity + str_->super_1->year + str_->doorsNumber;
}

int getFieldsSum_Buss(Buss* str_){
	return str_->super_1->capacity + str_->super_1->year + str_->super_2->doorsNumber + str_->ticketPrice;
}

void setTicketPrice_Buss_int(Buss* str_, int newPrice){
	str_->ticketPrice = newPrice;
}

int getTicketPrice_Buss(Buss* str_){
	return str_->ticketPrice;
}
```

---

## 👤 Author

> GitHub: [xrddev](https://github.com/xrddev)


## 📝 License

Released under the [MIT License](LICENSE). Originally built as part of a university project.
