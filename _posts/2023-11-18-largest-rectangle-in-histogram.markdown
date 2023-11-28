---
layout: post
title:  "Largest Rectangle in Histogram"
date:   2023-11-18 20:07:00 -0800
categories: javascript leetcode
---
So I ended up feeling more sick today than I did yesterday. Nevertheless, I decided to do a LeetCode
hard problem today. Let me explain.

When I'm sick, I actually have a hard time concentrating on passive media, such as reading or
watching something. I'll fall asleep, in fact. So I need to be doing something active. Thus, working on a programming problem is,
somewhat counterintuitively, a good activity for me when I'm sick.

I picked a problem I had encountered years before: [Largest Rectange in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/editorial/), a LeetCode hard. In fact, I first encountered this
problem in a phone screen for Facebook back in 2014. Which is funny: a LeetCode hard in a phone screen.
Gotta respect keeping the bar high. Back then, I hadn't done enough prep, so I largely got stuck in
the "dang this is hard" loop. Thus, I wanted to take another crack at it.

This time, my goals were fairly modest: arrive at a correct solution that is generally better than
the most naive/brute force. Being rusty, and under the weather, I was doubtful that I would get at the
optimal solution,
as the kind of mental "pattern matching" needed to find optimal solutions takes both a fresh brain
and enough practice.

Even for this at-home retry, I didn't get out a whiteboard or pad of paper, and ended up regretting it.
I can't really rapidly try out ideas and examples on a keyboard. Drawing and scribbling ideas is way
faster for me.

Ultimately, I achieved my modest goal. Here is my code. Not great, but not terrible.

{% highlight javascript %}
var largestRectangleArea = function(heights) {
    if (!heights || heights.length === 0) {
        throw new Error('heights len is guaranteed >= 1');
    }

    class Range {
        constructor(startIdx, endIdx, startHeight, endHeight, prevRange) {
            this.startIdx = startIdx;
            this.endIdx = endIdx;
            this.startHeight = startHeight;
            this.endHeight = endHeight;
            this.prevRange = prevRange;
        }
    }

    let maxArea = 0;
    const n = heights.length;
    let frontierRange = new Range(0, -1, (heights[0] === 0) ? 0 : 1, heights[0], null);

    for (let i = 1; i < n; i++) {
        let h = heights[i];

        if (h === frontierRange.endHeight) {
            // No change in height, so the prev range is still current.
            continue;
        } else if (h > frontierRange.endHeight) {
            // Start a new range.
            let r = new Range(i, -1, frontierRange.endHeight + 1, h, frontierRange);
            frontierRange.endIdx = i - 1;
            frontierRange = r;
        } else {
            // Find either the first range <= the new height, or the first range total.
            let prevs = [];
            let x = frontierRange;
            do {
                prevs.push(x);
                if (x.endHeight > h) {
                    x = x.prevRange;
                } else {
                    break;
                }
            } while (x);

            const leftmostRange = prevs.pop();
            if (leftmostRange.endHeight < h) {
                if (leftmostRange.endIdx === -1) {
                    throw new Error('If leftmost < h, there must be intermediate, hence leftmost should have endIdx');
                }

                // If strictly <, then we start a new range 1 to the right.
                const r = new Range(leftmostRange.endIdx + 1, -1, leftmostRange.endHeight + 1, h, leftmostRange);
                // The leftmostRange is still open, since it is < h. So don't compute area yet.
                // However, every other range was > h, so they're done.
                while (prevs.length > 0) {
                    const doneRange = prevs.pop();
                    const newArea = doneRange.endHeight*(i - doneRange.startIdx);
                    if (newArea > maxArea) {
                        maxArea = newArea;
                    }
                }
                frontierRange = r;
            } else if (leftmostRange.endHeight > h) {
                // If strictly >, then we start a new range at the very beginning but from the
                // current height. The original height of the start range is used below to compute
                // a possible area.
                if (leftmostRange.startIdx !== 0) {
                    throw new Error("Only the first range should be greater");
                }
                const r = new Range(0, -1, (h === 0) ? 0 : 1, h, null);

                // We want to get rid of all prior ranges, so re-enqueue leftmost.
                prevs.push(leftmostRange);
                while (prevs.length > 0) {
                    const doneRange = prevs.pop();
                    const newArea = doneRange.endHeight*(i - doneRange.startIdx);
                    if (newArea > maxArea) {
                        maxArea = newArea;
                    }
                }
                frontierRange = r;
            } else {
                // else, leftmostRange has same height. Since it is still open, we don't finalize
                // its area yet.
                while (prevs.length > 0) {
                    const doneRange = prevs.pop();
                    const newArea = doneRange.endHeight*(i - doneRange.startIdx);
                    if (newArea > maxArea) {
                        maxArea = newArea;
                    }
                }

                frontierRange = leftmostRange;
                leftmostRange.endIdx = -1;
            }
        }
    }

    while (frontierRange) {
        // height * (lastIdx - startIdx + 1)
        //     => height * (n - 1 - startIndex + 1)
        //     => height * (n - startIndex)
        const newArea = frontierRange.endHeight*(n - frontierRange.startIdx);
        if (newArea > maxArea) {
            maxArea = newArea;
        }
        frontierRange = frontierRange.prevRange;
    }

    return maxArea;
};
{% endhighlight javascript %}

That submission beat 32% on runtime and 67% on memory. But it's clearly too much code to
be an optimal solution. When you're writing too much code on well-traffic'd problems, it's usually a clue.

The motivating idea was a simple invariant: a monotonically nondecreasing histogram is easy to
score, so any time we have a decreasing bar, rewrite the histogram to be monotonically nondecreasing
by "closing off" any open rectangles to the left of the decrease. This was done by managing and
rewriting open "ranges".

Anyways, it was still a decent experience.