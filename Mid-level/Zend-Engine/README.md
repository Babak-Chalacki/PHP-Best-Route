<p><b>Day 1 👣</b></p>

<h3>⚙️ Zend Engine</h3>

<p>When we search this term on the web, we usually find this definition:</p>

<p><i>Zend Engine is the open-source, C-written core that serves as the compiler and runtime environment for the PHP scripting language.</i></p>

<br>
<p>But what really happens behind the scenes? 🤔 Let’s explore step by step.</p>

<p>The <b>Zend Engine</b> is the core of the PHP language — it <b>interprets and executes</b> your code and works like the <b>brain of PHP</b> 🧠.</p>

<p>Its main job is to convert your code into an executable form by following several stages, finally running it inside the virtual machine.</p>
<br>

<h4>1️⃣ Lexical Analysis</h4>
<p>At this stage, the code is broken down into <b>identifiable tokens</b>.</p>
<p>For example: <i>&lt;?php</i> → PHP Start Tag, <i>$sum</i> → Variable.</p>
<p>If there are any <b>syntax errors</b>, they are detected here ⚠️.</p>
<br>

<h4>2️⃣ Parsing</h4>
<p>Next, those tokens are turned into a <b>tree structure</b> known as the <i>Abstract Syntax Tree (AST)</i>.</p>
<p>Example code: <i>$x = 3 + 4;</i></p>
<p>The AST looks like this:</p>
<p>Assign<br>
├── Variable (<i>$x</i>)<br>
└── BinaryOperation (+)<br>
  ├── Literal (<i>3</i>)<br>
  └── Literal (<i>4</i>)</p>
<br>

<h4>3️⃣ Compilation</h4>
<p>Now, the AST is converted into <b>Opcode</b>. Opcode (short for Operation Code) is the intermediate code that the <i>Zend Virtual Machine</i> can execute.</p>
<p>Example:</p>
<p><i>0: ZEND_ASSIGN_VAR $x</i><br>
<i>1: ZEND_ADD 3, 4</i><br>
<i>2: ZEND_ASSIGN_OP $x, ZEND_ADD</i></p>
<br>

<h4>4️⃣ Execution</h4>
<p>Once the code is compiled into Opcode, the <b>Zend Virtual Machine</b> executes it line by line 💻.</p>
<p>The process goes like this:</p>
<p>- The opcodes are turned into machine-level instructions and executed.<br>
- The Zend VM manages memory and applies data changes.</p>
<p>During this process, <b>Garbage Collection</b> automatically cleans up unused memory ♻️.</p>
<br>

<h4>✨ Extra Notes</h4>
<p>- Zend Engine executes code much like a CPU would ⚡<br>
- Understanding these steps helps developers write <b>faster and more optimized PHP code</b> 💡</p>