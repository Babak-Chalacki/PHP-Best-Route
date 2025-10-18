<h3>🧠 Zend Memory Manager (ZMM) in PHP</h3>

<h3>🧠 Introduction: Why Does PHP Manage Memory Itself?</h3>

<p>
In low-level languages like C, you allocate memory from the operating system using <code>malloc()</code> 
and must free it manually using <code>free()</code>.
However, PHP (built on top of the Zend Engine) creates a dedicated <strong>memory pool</strong> for each request 
and manages it internally.
</p>

<p>This approach allows PHP to:</p>
<ul>
  <li>Allocate memory faster (since it uses a pool, not direct OS calls)</li>
  <li>Free all memory at once at the end of each request</li>
  <li>Reduce the chance of memory leaks</li>
</ul>

<hr>

<h3>⚙️ Section 1: Memory Pool in PHP</h3>

<p>When PHP executes a script:</p>
<ol>
  <li>A memory pool is created.</li>
  <li>All variables and internal structures (zvals, arrays, objects, etc.) are stored inside that pool.</li>
  <li>When the request ends, the entire pool is released.</li>
</ol>

<p>That means PHP never calls <code>malloc()</code> for every small variable. 
Instead, Zend uses its own internal functions:</p>

<table>
  <tr>
    <th>Function</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>emalloc()</code></td>
    <td>Allocate memory inside the Zend Memory Manager</td>
  </tr>
  <tr>
    <td><code>efree()</code></td>
    <td>Free allocated memory</td>
  </tr>
  <tr>
    <td><code>ecalloc()</code></td>
    <td>Allocate zero-initialized memory</td>
  </tr>
  <tr>
    <td><code>erealloc()</code></td>
    <td>Reallocate (resize) a memory block</td>
  </tr>
</table>

<hr>

<h3>💡 Section 2: Difference Between emalloc and malloc</h3>

<table>
  <tr>
    <th>Difference</th>
    <th><code>malloc()</code></th>
    <th><code>emalloc()</code></th>
  </tr>
  <tr>
    <td>Level</td>
    <td>Operating System</td>
    <td>Zend Engine</td>
  </tr>
  <tr>
    <td>Management</td>
    <td>By C programmer</td>
    <td>By Zend Memory Manager</td>
  </tr>
  <tr>
    <td>Speed</td>
    <td>Slower (system call)</td>
    <td>Faster (pool allocation)</td>
  </tr>
  <tr>
    <td>Cleanup</td>
    <td>Manual <code>free()</code></td>
    <td>Automatic at the end of request</td>
  </tr>
</table>

<hr>

<h3>🔍 Section 3: Request Lifecycle (Memory Lifetime in PHP)</h3>

<pre><code>&lt;?php
function run() {
    $a = range(1, 100000);
    $b = range(1, 100000);
}

run();
echo "done";
?&gt;
</code></pre>

<p>Explanation:</p>
<ul>
  <li>When <code>$a</code> is created, PHP gets memory from the pool using <code>emalloc()</code>.</li>
  <li>When <code>$b</code> is created, it allocates more memory from the same pool.</li>
  <li>When the function ends, refcounts reach zero and GC (Garbage Collector) clears them.</li>
  <li>When the script ends, Zend frees the entire memory pool.</li>
</ul>

<p><strong>Note:</strong> Even if the GC doesn’t free some objects, the memory is still released at the end of the request.  
Memory leaks in PHP only matter within the same request lifecycle.</p>

<hr>

<h3>🧩 Section 4: Persistent vs Non-Persistent Memory</h3>

<p>PHP has two main memory types:</p>

<table>
  <tr>
    <th>Type</th>
    <th>Lifetime</th>
    <th>Example</th>
    <th>Usage</th>
  </tr>
  <tr>
    <td>Non-persistent</td>
    <td>Only during the request</td>
    <td>Normal variables</td>
    <td>Default for scripts</td>
  </tr>
  <tr>
    <td>Persistent</td>
    <td>Shared across requests</td>
    <td>Opcode cache, extension data</td>
    <td>Used in C extensions or OPcache</td>
  </tr>
</table>

<p>For example, if an extension wants to store data between requests (like caching),  
it should use <code>pemalloc()</code> (persistent emalloc).</p>

<hr>

<h3>🔬 Section 5: Arena Allocation</h3>

<p>
Zend allocates memory in large chunks called <strong>arenas</strong>.
It requests a big block (e.g., 1 MB) from the OS, then splits it into smaller pieces for variables.
</p>

<p>Example concept:</p>
<ol>
  <li>Zend requests 1 MB from the OS.</li>
  <li>A variable needs 100 bytes → it takes 100 bytes from that block.</li>
  <li>When the block is full, a new one is allocated.</li>
  <li>At the end, all arenas are freed together.</li>
</ol>

<p>This design helps:</p>
<ul>
  <li>Reduce the number of system malloc/free calls</li>
  <li>Increase allocation speed</li>
  <li>Minimize memory fragmentation</li>
</ul>

<hr>

<h3>🧪 Section 6: Measuring and Analyzing Memory in PHP</h3>

<p>
PHP provides built-in functions to measure and analyze memory usage during script execution.  
These functions help developers monitor how much memory their code consumes and optimize performance.
</p>

<h4>📏 <code>memory_get_usage()</code></h4>
<p>
Returns the amount of memory currently allocated to the PHP script (in bytes).  
If you pass <code>true</code> as a parameter, it reports the real amount allocated from the operating system, 
not just Zend's internal pool.
</p>

<pre><code>&lt;?php
echo memory_get_usage() . "\n";        // Current memory usage
$a = range(1, 100000);
echo memory_get_usage() . "\n";        // After allocation
unset($a);
echo memory_get_usage() . "\n";        // After release
echo memory_get_usage(true) . "\n";    // Real usage from OS
?&gt;
</code></pre>

<p><b>Example Output:</b></p>
<pre><code>
360000
9000000
370000
380000
</code></pre>

<p>
The difference shows how the Zend Memory Manager keeps its internal pool 
even after releasing variables — improving performance by reusing memory blocks 
without repeatedly calling the operating system.
</p>

<hr>

<h4>📊 <code>memory_get_peak_usage()</code></h4>
<p>
Returns the peak (maximum) memory usage since the start of the script.  
This is useful for profiling heavy scripts such as data processing or loops.
</p>

<pre><code>&lt;?php
$a = range(1, 500000);
echo memory_get_peak_usage() . "\n"; // Peak memory used
unset($a);
echo memory_get_peak_usage() . "\n"; // Still shows max usage
?&gt;
</code></pre>

<p>
<code>memory_get_peak_usage()</code> keeps the highest recorded memory value, 
even after variables are freed, allowing you to analyze your script’s maximum footprint.
</p>

<hr>

<h3>💬 Section 7: Practical Notes</h3>

<h4>unset vs null</h4>
<pre><code>&lt;?php
$a = [1, 2, 3];
unset($a); // memory freed (refcount = 0)
$b = null; // refcount decreases, but variable still exists
?&gt;
</code></pre>

<h4>Using Generators to Save Memory</h4>
<pre><code>&lt;?php
function getItems() {
    foreach (range(1, 1000000) as $i) {
        yield $i;
    }
}

foreach (getItems() as $num) {
    // Only one item stays in memory at a time
}
?&gt;
</code></pre>

<h4>Array vs Object</h4>
<ul>
  <li>Arrays in PHP consume more memory because they use hash tables.</li>
  <li>Objects are lighter for structured data but have class overhead.</li>
</ul>

<hr>

<h3>🧠 Visual Summary</h3>

<table>
  <tr>
    <th>Concept</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Zend Memory Manager</td>
    <td>Internal PHP memory allocation system</td>
  </tr>
  <tr>
    <td>emalloc / efree</td>
    <td>Zend’s replacement for malloc/free</td>
  </tr>
  <tr>
    <td>Memory Pool</td>
    <td>Temporary memory reserved per request</td>
  </tr>
  <tr>
    <td>Persistent Memory</td>
    <td>Memory that persists across requests (e.g., extensions)</td>
  </tr>
  <tr>
    <td>Arena Allocator</td>
    <td>Allocates large memory blocks and splits them efficiently</td>
  </tr>
  <tr>
    <td>Request Cleanup</td>
    <td>All memory released at the end of request</td>
  </tr>
  <tr>
    <td>memory_get_usage()</td>
    <td>Shows current memory usage</td>
  </tr>
  <tr>
    <td>memory_get_peak_usage()</td>
    <td>Shows the highest memory used during execution</td>
  </tr>
</table>

<hr>

<p><strong>Conclusion:</strong>  
Zend Memory Manager (ZMM) is one of the core systems that make PHP efficient and safe to use, 
automatically handling memory allocation, optimization, and cleanup for every request.</p>
