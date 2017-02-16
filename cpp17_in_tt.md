Decomposition Declarations a.k.a Structured Bindings 
-------------------

<table>
<tr>
<th>
C++14
</th>
<th>
C++14
</th>
<th>
C++17
</th>
</tr>
<tr>
<td  valign="top">
<pre lang="cpp">
   tuple&lt;int, string&gt; stuff();
   
   auto tup = stuff();
   int i = get&lt;0&gt;(tup);
   string s = get&lt;1&gt;(tup);
  
   use(s, i);
</pre>
</td>
<td  valign="top">
<pre lang="cpp">
   tuple&lt;int, string&gt; stuff();
   
   int i;
   string s;
   std::tie(i,s) = stuff();

   use(s, i);
</pre>
</td>
<td valign="top">
<pre lang="cpp">
   tuple&lt;int, string&gt; stuff();
   
   
   auto [ i, s ] = stuff();


   use(s, i);
</pre>
</td>
</tr>
</table>



<table>
<tr>
<th>
C++17
</th>
<th>
compiler
</th>
</tr>
<tr>
<td valign="top">
<pre lang="cpp">
   pair&lt;int, string&gt; stuff();
   
   
   auto &amp; [ i, s ] = stuff();


   use(s, i);
</pre>
</td>
<td valign="top">
<pre lang="cpp">
   pair&lt;int, string&gt; stuff();
   
   auto &amp; __tmp = stuff();
   auto &amp; i = get&lt;0&gt;(__tmp);
   auto &amp; s = get&lt;1&gt;(__tmp);

   use(s, i);
</pre>
</td>
</tr>
</table>





<table>
<tr>
<th>
C++17
</th>
</tr>
<tr>
<td valign="top">
<pre lang="cpp">
   pair&lt;int, string&gt; stuff();
   
   
   auto &amp;&amp; [ i, s ] = stuff();


   use(s, i);
</pre>
</td>
<td valign="top">
<pre lang="cpp">
   pair&lt;int, string&gt; stuff();
   
   // ???
   auto &amp;&amp; __tmp = stuff();
   auto &amp;&amp; i = get&lt;0&gt;(__tmp);
   auto &amp;&amp; s = get&lt;1&gt;(__tmp);

   use(s, i);
</pre>
</td>
</tr>
</table>
