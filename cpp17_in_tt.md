Decomposition Declarations a.k.a Structured Bindings 
-------------------

<table>
<tr>
<th>
C++14
</td>
<th>
C++14
</td>
<th>
C++17
</td>
</tr>
<tr>
<td  valign="top">
<pre lang="cpp">
   tuple&lt;int, string&gt; stuff();
   
   auto tup = stuff();
   int i = get<0>(tup);
   string s = get<1>(tup);
  
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
