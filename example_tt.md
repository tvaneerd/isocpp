## Going Towards a More General Looping Mechanism

Document number: DXXXXR0  
Date: 20XX-MM-DD  
Audience: EWG  
Reply-to: Tony Van Eerd. tt at forecode.com


### Summary

Current looping mechanisms (`for`loops, `while` and `do`...`while` loops)
are great - in their niche uses;
but they are often found wanting, resulting in awkward constructs.
In particular, in a few cases it is impossible to NOT repeat yourself (ie a violation of DRY) or to be concise, using those mechanisms.

Looking at how all these mechanisms work, we can refactor them into a single, more general, control flow mechanism.
In fact, it could *replace* all the other keywords with this single mechanism!

_(Note, for the sake of backwards compatibility, we do not recommend deprecation at this time.
If the new style is widely adopted - as we expect it will - we can deprecate the old keywords
when common coding practices show they have fallen out of favour.)_



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

... show more problematic uses of current control flow, compared against new mechanism

### Proof of Replacement

... show before/after table replacing each of { `for`, `while` and `do`...`while` }

### Technical Details or Preliminary Wording, Etc

Label statements, goto statement. Etc.

### Further Work

Initial results suggest a *"computed goto"* could also replace `if` and `switch` statements - thus completely **replacing all control flow mechanisms** with a single keyword.  The committee should *give me* a kidney.
