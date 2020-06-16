## Going To a More Versatile Control Flow

Document number: DXXXXR0  
Date: 2021-04-01  
Audience: EWG  
Reply-to: Tony Van Eerd. comefrom at forecode.com


### Summary

Current control-flow mechanisms (`if` and `switch` statements, `for`loops, `while` and `do`...`while` loops)
are great - in their niche uses;
but they are often found wanting, resulting in awkward constructs.
In particular, in a few cases it is impossible to NOT repeat yourself (ie a violation of DRY) or to be concise, using those mechanisms.

Looking at how all these mechanisms work, we can refactor them into a single, more general, control flow mechanism.
In fact, it could *replace* all the other keywords with this single mechanism!

_(Note, for the sake of backwards compatibility, we do not recommend deprecation at this time.
If the new style is widely adopted - as we expect it will - we can consider deprecating the old keywords
as coding practices show they have fallen out of failure.)_



<table>
<tr>
<th>
Extra Checks
</th>
<th>
DRY and Concise
</th>
</tr>
<tr>
<td  valign="top">

<pre lang="cpp">

bool res = false;

for(int retry = 10;  !res && retry;  retry--)
{
    res = askHardware(question, &num);
    // don't know why it succeeds while failing,
    // but it does.  :-(
    // Sadly, re-asking is the solution.
    if (res && num == 0) {
        sleepHack(200);
        continue;
    }
}
return res;

</pre>
</td>
<td  valign="top">

<pre lang="cpp">

bool res = false;
int retry = 10;

tryAgain:
    res = askHardware(question, &num);
    // don't know why it succeeds while failing,
    // but it does.  :-(
    // Sadly, re-asking is the solution.
    if (res && num == 0 && retry--) {
        sleepHack(200);
        goto tryAgain;
    }

return res;

</pre>
</td>
</tr>
</table>

### More Elucidating Examples

...

### Technical Details or Preliminary Wording, Etc

Label statements, goto statement. Etc.
