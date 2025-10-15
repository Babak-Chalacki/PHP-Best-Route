<p><b>Day 2 👣</b></p>

<h3>⚙️ PHP Garbage Collector (GC)</h3>

<h4>🧠 What Is the Garbage Collector?</h4>
<p>
The Garbage Collector (GC) in PHP is a built-in system in the Zend Engine responsible for automatic memory management.
It finds and frees memory used by variables that are no longer needed, especially cyclic references that normal reference counting cannot handle.
</p>

<h4>🔍 How Memory Management Normally Works</h4>
<p>
PHP uses a reference counting system to manage memory.
Every variable is stored in a structure called <code>zval</code>, which contains:
</p>

<ul>
  <li>The actual value</li>
  <li>The type (string, int, object, etc.)</li>
  <li>A reference count (<code>refcount</code>)</li>
  <li>Some flags</li>
</ul>

<pre><code>$a = "Hello";
$b = $a;
</code></pre>

<p>Both <code>$a</code> and <code>$b</code> point to the same zval, so <code>refcount = 2</code>.</p>

<pre><code>unset($a);
</code></pre>

<p>
Now <code>refcount = 1</code>.  
When <code>$b</code> is also unset, <code>refcount</code> becomes 0 and PHP frees the memory.
</p>

<h4>⚠️ The Problem: Cyclic References</h4>
<p>
Sometimes objects reference each other, forming a cycle that prevents <code>refcount</code> from reaching 0, which causes a memory leak.
</p>

<pre><code>$a = new stdClass();
$b = new stdClass();

$a->b = $b;
$b->a = $a;

unset($a, $b);
</code></pre>

<p>
Even after unsetting, both objects still reference each other, so their memory is not released.
</p>

<h4>♻️ The Solution: Garbage Collector</h4>
<p>
The Garbage Collector scans memory to detect cyclic references and free them.
It is triggered when:
</p>

<ol>
  <li>The number of tracked variables exceeds a threshold.</li>
  <li>You manually call <code>gc_collect_cycles();</code></li>
</ol>

<p>You can control GC behavior with these functions:</p>

<pre><code>gc_enable();         // Enable GC
gc_disable();        // Disable GC
gc_enabled();        // Check GC status
gc_collect_cycles(); // Force garbage collection
</code></pre>

<h4>🧩 How GC Works Internally</h4>
<ol>
  <li><b>Root Buffering:</b> Zend Engine keeps a list of possible cyclic variables in a root buffer.</li>
  <li><b>Mark Phase:</b> Marks all variables that are still reachable.</li>
  <li><b>Sweep Phase:</b> Frees all variables that are not reachable.</li>
</ol>

<p>Implementation files in PHP source:</p>
<pre><code>Zend/zend_gc.c
Zend/zend_gc.h
Zend/zend_types.h
</code></pre>

<h4>⚙️ Example of Manual GC Usage</h4>
<pre><code>gc_disable(); // Turn off automatic GC

for ($i = 0; $i < 10000; $i++) {
    $a = new stdClass();
    $b = new stdClass();
    $a->b = $b;
    $b->a = $a;
}

echo gc_collect_cycles(); // Run GC manually
gc_enable(); // Re-enable GC
</code></pre>

<h4>📊 Garbage Collector Functions</h4>
<table>
  <tr>
    <th>Function</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>gc_enable()</code></td>
    <td>Enable the Garbage Collector</td>
  </tr>
  <tr>
    <td><code>gc_disable()</code></td>
    <td>Disable the Garbage Collector</td>
  </tr>
  <tr>
    <td><code>gc_enabled()</code></td>
    <td>Check if GC is active</td>
  </tr>
  <tr>
    <td><code>gc_collect_cycles()</code></td>
    <td>Run GC manually and return number of freed cycles</td>
  </tr>
</table>

<h4>🧠 Summary</h4>
<table>
  <tr>
    <th>Concept</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Reference Counting</td>
    <td>Frees memory when refcount = 0</td>
  </tr>
  <tr>
    <td>Cyclic References</td>
    <td>Cause memory leaks because refcount never hits 0</td>
  </tr>
  <tr>
    <td>Garbage Collector</td>
    <td>Detects and removes cyclic references</td>
  </tr>
  <tr>
    <td>Source Files</td>
    <td>zend_gc.c, zend_gc.h, zend_types.h</td>
  </tr>
  <tr>
    <td>Manual Control</td>
    <td>Use gc_enable(), gc_disable(), gc_collect_cycles()</td>
  </tr>
</table>
