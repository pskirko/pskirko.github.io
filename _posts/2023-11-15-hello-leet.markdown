I started by retyping yesterday’s code without looking anything up; all good.

I then picked a LeetCode question out of the question list at the end of Chapter 1 in Skiena’s
Algorithm Design Manual (3rd Ed): [leetcode.com/problems/rotate-list](https://leetcode.com/problems/rotate-list).

Here is my solution. I didn’t write it straight like this; initially, I was going to reverse the
list to more easily perform the rotation, then I realized I don’t need to perform each rotation,
I can just perform fixups.

{% highlight javascript %}
var rotateRight = function(head, k) {
   if (!head) { return head; }
   let nodes = [];
   let curr = head;
   while (curr !== null) {
       nodes.push(curr);
       curr = curr.next;
   }

   let cnt = nodes.length;
   k %= cnt;
   if (k === 0) {
       return head;
   }

   let newHeadIdx = cnt - k;
   nodes[newHeadIdx - 1].next = null;
   nodes[cnt-1].next = nodes[0];
   return nodes[newHeadIdx];
};
{% endhighlight javascript%}

Also note I missed checking for k === 0 at the top, oops a little rusty, but ultimately not a big
deal.

That said, I mostly aimed for simplicity and brevity of code, and time optimality. Pretty happy with
the result, as it’s short and easy to understand.
