# The Interpreter Pattern: Building Your Own Mini Language

Ever written a formula in Excel? Ever used regular expressions? Ever configured routes in a web framework? You've used languages interpreted at runtime. That's the Interpreter pattern.

The idea: define a grammar for a language and build an interpreter that processes sentences in that language.

## The Problem: Hardcoded Logic for Expressions

You're building a calculator that needs to evaluate expressions:

```java
class Calculator {
    public int evaluate(String expression) {
        if (expression.equals("5 + 3")) {
            return 8;
        } else if (expression.equals("10 - 2")) {
            return 8;
        } else if (expression.equals("4 * 2")) {
            return 8;
        }
        // Add every possible expression? Impossible!
    }
}
```

Problems:

**Can't handle new expressions:** Need to hardcode every possible expression.

**Not scalable:** Can't add new operations without modifying code.

**No grammar:** Can't define rules for valid expressions.

**Not flexible:** Can't combine operations like "(5 + 3) * 2".

What if we could define a grammar and build an interpreter to evaluate any expression?

## The Interpreter Solution

Define a grammar and create classes to interpret it:

```java
// Abstract expression
interface Expression {
    int interpret();
}

// Terminal expression: number
class NumberExpression implements Expression {
    private int number;
    
    public NumberExpression(int number) {
        this.number = number;
    }
    
    @Override
    public int interpret() {
        return number;
    }
}

// Non-terminal expression: addition
class AddExpression implements Expression {
    private Expression left;
    private Expression right;
    
    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() + right.interpret();
    }
}

// Non-terminal expression: subtraction
class SubtractExpression implements Expression {
    private Expression left;
    private Expression right;
    
    public SubtractExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() - right.interpret();
    }
}

// Non-terminal expression: multiplication
class MultiplyExpression implements Expression {
    private Expression left;
    private Expression right;
    
    public MultiplyExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() * right.interpret();
    }
}
```

Usage:

```java
// Build expression tree for: (5 + 3) * 2
Expression expression = new MultiplyExpression(
    new AddExpression(
        new NumberExpression(5),
        new NumberExpression(3)
    ),
    new NumberExpression(2)
);

int result = expression.interpret();
System.out.println("Result: " + result);  // 16
```

The expression is represented as a tree. Calling `interpret()` evaluates it recursively.

## The Components

### 1. Abstract Expression

Declares interpret operation:

```java
interface Expression {
    int interpret();
}
```

### 2. Terminal Expression

Implements interpret for terminal symbols (leaves):

```java
class NumberExpression implements Expression {
    private int value;
    
    public int interpret() {
        return value;
    }
}
```

### 3. Non-Terminal Expression

Implements interpret for grammar rules (operators):

```java
class AddExpression implements Expression {
    private Expression left;
    private Expression right;
    
    public int interpret() {
        return left.interpret() + right.interpret();
    }
}
```

### 4. Context (Optional)

Holds global information:

```java
class Context {
    private Map<String, Integer> variables;
    
    public void setVariable(String name, int value) {
        variables.put(name, value);
    }
    
    public int getVariable(String name) {
        return variables.get(name);
    }
}
```

## Real-World Example: Boolean Expression Evaluator

```java
// Abstract expression
interface BooleanExpression {
    boolean interpret(Context context);
}

// Terminal: variable
class VariableExpression implements BooleanExpression {
    private String name;
    
    public VariableExpression(String name) {
        this.name = name;
    }
    
    @Override
    public boolean interpret(Context context) {
        return context.lookup(name);
    }
}

// Terminal: constant
class ConstantExpression implements BooleanExpression {
    private boolean value;
    
    public ConstantExpression(boolean value) {
        this.value = value;
    }
    
    @Override
    public boolean interpret(Context context) {
        return value;
    }
}

// Non-terminal: AND
class AndExpression implements BooleanExpression {
    private BooleanExpression left;
    private BooleanExpression right;
    
    public AndExpression(BooleanExpression left, BooleanExpression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public boolean interpret(Context context) {
        return left.interpret(context) && right.interpret(context);
    }
}

// Non-terminal: OR
class OrExpression implements BooleanExpression {
    private BooleanExpression left;
    private BooleanExpression right;
    
    public OrExpression(BooleanExpression left, BooleanExpression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public boolean interpret(Context context) {
        return left.interpret(context) || right.interpret(context);
    }
}

// Non-terminal: NOT
class NotExpression implements BooleanExpression {
    private BooleanExpression expression;
    
    public NotExpression(BooleanExpression expression) {
        this.expression = expression;
    }
    
    @Override
    public boolean interpret(Context context) {
        return !expression.interpret(context);
    }
}

// Context
class Context {
    private Map<String, Boolean> variables = new HashMap<>();
    
    public void assign(String name, boolean value) {
        variables.put(name, value);
    }
    
    public boolean lookup(String name) {
        return variables.getOrDefault(name, false);
    }
}
```

Usage:

```java
Context context = new Context();
context.assign("x", true);
context.assign("y", false);

// Expression: (x AND NOT y) OR y
BooleanExpression expression = new OrExpression(
    new AndExpression(
        new VariableExpression("x"),
        new NotExpression(new VariableExpression("y"))
    ),
    new VariableExpression("y")
);

boolean result = expression.interpret(context);
System.out.println("Result: " + result);  // true
```

## Another Example: SQL-Like Query Language

```java
// Grammar:
// SELECT [columns] FROM [table] WHERE [condition]

interface QueryExpression {
    List<Map<String, Object>> interpret(Database db);
}

class SelectExpression implements QueryExpression {
    private List<String> columns;
    private String table;
    private ConditionExpression condition;
    
    public SelectExpression(List<String> columns, String table, 
                           ConditionExpression condition) {
        this.columns = columns;
        this.table = table;
        this.condition = condition;
    }
    
    @Override
    public List<Map<String, Object>> interpret(Database db) {
        List<Map<String, Object>> allRows = db.getTable(table);
        List<Map<String, Object>> filtered = new ArrayList<>();
        
        for (Map<String, Object> row : allRows) {
            if (condition == null || condition.evaluate(row)) {
                Map<String, Object> selectedRow = new HashMap<>();
                for (String column : columns) {
                    selectedRow.put(column, row.get(column));
                }
                filtered.add(selectedRow);
            }
        }
        
        return filtered;
    }
}

interface ConditionExpression {
    boolean evaluate(Map<String, Object> row);
}

class EqualsCondition implements ConditionExpression {
    private String column;
    private Object value;
    
    public EqualsCondition(String column, Object value) {
        this.column = column;
        this.value = value;
    }
    
    @Override
    public boolean evaluate(Map<String, Object> row) {
        return value.equals(row.get(column));
    }
}

class AndCondition implements ConditionExpression {
    private ConditionExpression left;
    private ConditionExpression right;
    
    public AndCondition(ConditionExpression left, ConditionExpression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public boolean evaluate(Map<String, Object> row) {
        return left.evaluate(row) && right.evaluate(row);
    }
}
```

Usage:

```java
Database db = new Database();
// Populate database...

// SELECT name, age FROM users WHERE age = 25 AND active = true
QueryExpression query = new SelectExpression(
    Arrays.asList("name", "age"),
    "users",
    new AndCondition(
        new EqualsCondition("age", 25),
        new EqualsCondition("active", true)
    )
);

List<Map<String, Object>> results = query.interpret(db);
```

## Parsing Strings into Expression Trees

Building expression trees manually is tedious. Usually you parse strings:

```java
class ExpressionParser {
    public Expression parse(String input) {
        // Tokenize
        String[] tokens = input.split(" ");
        
        // Build expression tree
        Stack<Expression> stack = new Stack<>();
        Stack<String> operators = new Stack<>();
        
        for (String token : tokens) {
            if (isNumber(token)) {
                stack.push(new NumberExpression(Integer.parseInt(token)));
            } else if (isOperator(token)) {
                while (!operators.isEmpty() && 
                       precedence(operators.peek()) >= precedence(token)) {
                    Expression right = stack.pop();
                    Expression left = stack.pop();
                    String op = operators.pop();
                    stack.push(createExpression(op, left, right));
                }
                operators.push(token);
            }
        }
        
        while (!operators.isEmpty()) {
            Expression right = stack.pop();
            Expression left = stack.pop();
            String op = operators.pop();
            stack.push(createExpression(op, left, right));
        }
        
        return stack.pop();
    }
    
    private Expression createExpression(String op, Expression left, Expression right) {
        switch (op) {
            case "+": return new AddExpression(left, right);
            case "-": return new SubtractExpression(left, right);
            case "*": return new MultiplyExpression(left, right);
            default: throw new IllegalArgumentException("Unknown operator: " + op);
        }
    }
    
    private boolean isNumber(String token) {
        try {
            Integer.parseInt(token);
            return true;
        } catch (NumberFormatException e) {
            return false;
        }
    }
    
    private boolean isOperator(String token) {
        return token.equals("+") || token.equals("-") || token.equals("*");
    }
    
    private int precedence(String op) {
        if (op.equals("*")) return 2;
        if (op.equals("+") || op.equals("-")) return 1;
        return 0;
    }
}
```

Usage:

```java
ExpressionParser parser = new ExpressionParser();
Expression expr = parser.parse("5 + 3 * 2");
System.out.println(expr.interpret());  // 11
```

## When to Use Interpreter

Use Interpreter when:
- You have a simple grammar to interpret
- Efficiency is not critical (interpreter can be slow)
- Grammar doesn't change often
- You're building a domain-specific language (DSL)

Real scenarios:
- Configuration languages
- Query languages
- Rule engines
- Expression evaluators
- Regular expressions
- Template engines
- Math calculators
- Command processors

## When NOT to Use Interpreter

Don't use Interpreter when:
- Grammar is complex (use parser generators instead)
- Performance is critical (compiled approaches are faster)
- Grammar changes frequently (maintaining expression classes is tedious)

For complex grammars, consider:
- ANTLR (parser generator)
- JavaCC (Java Compiler Compiler)
- Existing expression libraries

## Interpreter in Real Frameworks

### Spring Expression Language (SpEL)

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')");
String message = (String) exp.getValue();
```

### Java Regex

```java
Pattern pattern = Pattern.compile("[a-z]+");
Matcher matcher = pattern.matcher("hello");
// Regex engine interprets the pattern
```

### SQL

Every database interprets SQL queries using the Interpreter pattern internally.

## Common Pitfalls

### Pitfall 1: Complex Grammar

```java
// BAD: Too many expression classes for complex grammar
class Expression1 { }
class Expression2 { }
// ... 50 more expression classes
```

For complex grammars, use parser generators.

### Pitfall 2: Mutable Expressions

```java
// BAD: Reusing expressions with changing state
Expression expr = parser.parse("x + y");
context.set("x", 5);
expr.interpret(context);  // 5 + y
context.set("x", 10);
expr.interpret(context);  // 10 + y (reusing same expression)
```

Expressions should be immutable or context should be passed fresh each time.

### Pitfall 3: No Caching

```java
// BAD: Parsing same expression repeatedly
for (int i = 0; i < 1000; i++) {
    Expression expr = parser.parse("5 + 3");  // Parsing every time!
    expr.interpret();
}

// GOOD: Parse once, reuse
Expression expr = parser.parse("5 + 3");
for (int i = 0; i < 1000; i++) {
    expr.interpret();
}
```

## Interpreter and Composite Pattern

Interpreter often uses Composite pattern:

**Composite structure:** Expression tree is a composite (non-terminals contain terminals).

**Recursive interpretation:** `interpret()` calls are recursive, like Composite operations.

## Interpreter and SOLID Principles

### Single Responsibility Principle

Each expression class handles one operation.

### Open/Closed Principle

Add new expression types without modifying existing ones.

## The Mental Model

Think of Interpreter like:

**Math teacher evaluating expressions:** Student writes "2 + 3 * 4". Teacher breaks it into parts: "3 * 4 = 12", then "2 + 12 = 14". Each operation has rules.

**Recipe instructions:** Recipe says "Mix A and B, then add C." Chef interprets each step. Each instruction (mix, add, bake) has meaning.

**Music notation:** Musicians read sheet music (grammar). Each symbol (notes, rests, dynamics) has meaning. Musicians interpret the score.

## Performance Considerations

Interpreter can be slow:
- Building expression tree takes time
- Recursive interpretation has overhead
- No optimization

Optimizations:
- Cache parsed expressions
- Compile to bytecode
- Use visitor pattern for multiple operations
- Consider JIT compilation for hot paths

## Testing with Interpreter

```java
@Test
public void testAddExpression() {
    Expression expr = new AddExpression(
        new NumberExpression(5),
        new NumberExpression(3)
    );
    assertEquals(8, expr.interpret());
}

@Test
public void testComplexExpression() {
    // (5 + 3) * 2
    Expression expr = new MultiplyExpression(
        new AddExpression(
            new NumberExpression(5),
            new NumberExpression(3)
        ),
        new NumberExpression(2)
    );
    assertEquals(16, expr.interpret());
}

@Test
public void testParser() {
    ExpressionParser parser = new ExpressionParser();
    Expression expr = parser.parse("5 + 3 * 2");
    assertEquals(11, expr.interpret());
}
```

## Alternative: Visitor Pattern

For multiple operations on the same grammar, use Visitor:

```java
interface ExpressionVisitor {
    int visitNumber(NumberExpression expr);
    int visitAdd(AddExpression expr);
    int visitMultiply(MultiplyExpression expr);
}

class EvaluationVisitor implements ExpressionVisitor {
    public int visitNumber(NumberExpression expr) {
        return expr.getValue();
    }
    
    public int visitAdd(AddExpression expr) {
        return expr.getLeft().accept(this) + expr.getRight().accept(this);
    }
    
    // ...
}

class PrintVisitor implements ExpressionVisitor {
    public int visitAdd(AddExpression expr) {
        System.out.println("Add operation");
        expr.getLeft().accept(this);
        expr.getRight().accept(this);
        return 0;
    }
    
    // ...
}
```

Visitor is better when you have many operations but stable grammar.

## Final Thoughts

The Interpreter pattern is about defining a language and building an interpreter for it. It's powerful for simple grammars but gets unwieldy for complex ones.

It's not about showing off. It's about:
- Defining domain-specific languages
- Evaluating expressions at runtime
- Creating flexible rule engines
- Building query languages

The key insight: represent grammar as classes. Interpret by traversing the structure.

Remember:
- Terminal expressions are leaves
- Non-terminal expressions are operations
- Context holds global state
- Keep grammar simple

Next time you need to evaluate expressions or rules at runtime, consider Interpreter. But know when to switch to parser generators for complex grammars.

Interpret wisely.