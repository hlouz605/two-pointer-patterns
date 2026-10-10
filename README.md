# two pointer technique: How Converging and Fast-Slow Pointers Turn O(n²) Loops into O(n), and Where to Practice Them

Most people meet the two pointer technique the same way: they write a nested loop, get a time limit exceeded verdict, and then someone says "just use two pointers." That advice is correct and completely useless on its own. The technique isn't a trick you apply to a problem — it's a loop invariant you maintain, and the hard part is knowing which of the three pointer families fits the problem in front of you.

This is a walkthrough of what actually makes two pointers work, the cases where it silently breaks, and how to practice it as a pattern rather than memorizing individual solutions.

## What the two pointer technique actually is

Two indices into a linear structure, moved according to a rule that guarantees you never have to revisit an element. The payoff is concrete: a brute-force pair search runs in O(n²) time, the pointer version runs in O(n) after an O(n log n) sort, with O(1) extra space because you're only storing two integers.

That space bound is the part people underestimate. A hash map solves many of the same problems in O(n) time — but it costs O(n) memory and, in interviews, it often gets you a follow-up question you can't answer. Two pointers is what you reach for when the interviewer says "can you do it without extra memory?"

The technique applies to any iterable with order, not just arrays. Linked lists, strings, and sorted matrices all show up in classic two-pointer problems. One common misconception worth clearing up early: two pointers is not "only for arrays." Cycle detection in a linked list, for example, is a fast-slow pointer problem and is genuinely O(1) space.

## The three families, and how to tell them apart

Almost every two-pointer problem falls into one of three shapes. Mixing them up is the most common source of wrong answers.

| Family | Pointer setup | Movement rule | Typical problems | Prerequisite |
| --- | --- | --- | --- | --- |
| Opposite-direction (converging) | `left = 0`, `right = n - 1` | Move the pointer whose side can't improve the current answer | Two Sum on sorted input, 3Sum, Valid Palindrome, Container With Most Water | Sorted or monotonic ordering |
| Fast and slow | Both start at the same end | `fast` advances faster; `slow` only advances when a condition holds | Remove Duplicates (in place), linked list cycle detection, find middle node | Usually in-place constraints |
| Same-direction (window) | `left = 0`, `right = 0` | Expand `right`, then shrink `left` while the window is invalid | Minimum Size Subarray Sum, Longest Substring Without Repeating Characters | A condition you can maintain incrementally |

The sliding window pattern is really a two-pointer technique with a constraint attached — which is why the two topics bleed into each other in every curriculum. If your pointers move in the same direction and you maintain a running aggregate over the range between them, you're doing a window. If they close in from both ends, you're converging.

The decision question that separates them: does the problem ask about a **pair** (converging) or a **contiguous range** (window)? Get that wrong and you'll spend twenty minutes fighting your own invariant.

## Why sorting is usually step zero — and what it costs

Converging pointers only work because order gives you information. If `nums[left] + nums[right]` is too small, and the array is sorted ascending, then `nums[left]` can't pair with anything to the right of `right` to hit the target. That element is dead. You advance `left` and never look back.

Without sorting, that argument collapses. You can't discard anything, because you have no idea what's further along.

So the standard shape of a two-pointer solution is: sort (O(n log n)), then one linear pass (O(n)). Total complexity is dominated by the sort.

The cost is real and worth saying out loud: **sorting destroys original indices.** If the problem asks you to return positions rather than values — "return the indices of the two numbers" — sorting the array directly breaks the answer. You need to sort index-value pairs, or keep a mapping back to the original positions. Plenty of people write a correct pointer loop and then fail on the return type.

## Worked example: pair sum on a sorted array


left, right = 0, len(nums) - 1
while left < right:
    s = nums[left] + nums[right]
    if s == target:
        return [left, right]
    elif s < target:
        left += 1    # nums[left] is too small to pair with anything
    else:
        right -= 1   # nums[right] is too large to pair with anything
return []


Three lines of logic, one invariant: everything outside `[left, right]` has been ruled out. Each iteration eliminates exactly one element, so the loop runs at most n times. No element is examined twice. That's the whole technique in miniature.

Notice what the branches mean. When the sum is too large, you can't move `left` — that would only make the sum smaller and you'd be discarding a pair you haven't disproven. The direction of movement is forced by the sortedness, not chosen freely. If you ever find yourself picking a pointer to move without being able to justify it in one sentence, your invariant is probably broken.

### Reducing 3Sum to Two Sum

3Sum is where this pattern earns its keep. Brute force is O(n³). The standard solution:

1. Sort the array.
2. Fix one element `nums[i]` in an outer loop.
3. For the remaining subarray, find a pair summing to `-nums[i]` using converging pointers.
4. Skip duplicates at all three positions.

That's O(n²) — one factor of n removed by the pointer pass. The duplicate skipping is where most submissions actually fail, and it's a detail the pointer template doesn't handle for you. After recording a valid triplet, you must advance `left` past equal values and retreat `right` past equal values, plus skip repeated values of `i` in the outer loop. Miss any of those and you return duplicate triplets.

## Worked example: in-place duplicate removal

Fast-slow pointers solve a different family of problems where the answer is "compress this array without allocating a new one."


slow = 0
for fast in range(1, len(nums)):
    if nums[fast] != nums[slow]:
        slow += 1
        nums[slow] = nums[fast]
return slow + 1


`slow` marks the end of the deduplicated prefix; `fast` scans ahead. Every element is read once. Same O(n) / O(1) profile, completely different movement rule from the converging version. If you try to solve this with `left = 0, right = n - 1`, you'll get nowhere — the pointers aren't converging toward an answer, one is chasing the other.

## The mistakes that cost interviews

**A pointer that doesn't move.** Any branch that neither advances `left` nor retreats `right` produces an infinite loop. This is the single most common two-pointer bug, and it usually hides in an `else` clause.

**`left < right` versus `left <= right`.** Ask what a single remaining element means for your problem. For pair-sum you stop at `left == right` because a pair needs two distinct indices. For a palindrome check you do want to examine the middle character. For boat rescue you must handle the last person standing, so `<=` is correct. The loop condition is a statement about the problem, not a style choice.

**Forcing pointers onto unsorted input.** If the array isn't sorted and you aren't allowed to sort it, two pointers may be the wrong tool. Unsorted Two Sum wants a hash map. The technique needs monotonic structure to prune with.

**Losing track of what's been proven.** Every pointer move should eliminate elements permanently. If a move doesn't eliminate anything, the amortized O(n) argument is gone and you're back to quadratic behavior with extra steps.

**Assuming two pointers always beats a hash map.** It doesn't. Two pointers wins on space, and on sorted-input follow-ups. A hash map wins when order doesn't exist and you can't create it.

> A quick self-test before you commit to the pattern: can you state, in one sentence, why the element you're about to discard can never be part of the answer? If not, you're guessing at pointer movement.

## A worked mental checklist for unfamiliar problems

When a problem mentions arrays or strings, run this in order:

1. Is the input sorted, or does sorting preserve what the problem asks for?
2. Does the problem ask for a pair, a triplet, or a decision about two endpoints? → converging pointers.
3. Does it ask about a contiguous subarray or substring under some constraint? → sliding window.
4. Does it require in-place modification or O(1) space? → fast-slow pointers.
5. Is the data unsorted, unsortable, and index-sensitive? → hash map.
6. Does the condition lack monotonicity (moving a pointer can't tell you which direction is correct)? → two pointers won't work.

That last point matters. Two pointers relies on a monotonic relationship: moving one pointer in one direction makes the quantity move predictably. When that relationship doesn't exist, no amount of cleverness saves the approach.

## Practicing the pattern instead of the problems

The reason two pointers feels obvious in retrospect and invisible in the moment is that recognition is the bottleneck, not implementation. The loop is five lines. Knowing that a problem is a two-pointer problem, in the first two minutes of an interview, is the skill being tested.

This is the gap AlgoMonster was built around. The platform is organized by coding patterns rather than by problem, and its own ranking page lists depth-first search, breadth-first search, and two pointers as the highest-return topics — by its data, those three make up roughly half of the problems that show up in interviews. If you only have time to solidify a handful of patterns, two pointers is on the list.

The two-pointer module itself (introduced with a short quiz that tests whether you actually understand applicability — for example, whether the technique works on linked lists, not just arrays) is followed by problems that shift through the variations: converging pointers on sorted input, in-place duplicate removal with fast-slow, and three-sum as the combination case.

What makes the format different from problem grinding:

- **Pattern selection comes before code.** For each problem you choose the pattern first, and the platform tells you whether your choice was right and why.
- **Templates per pattern, in multiple languages.** You get the generalized implementation, not one problem's answer, so you can adapt it to variants.
- **Speedrun mode.** Rapid-fire multiple choice on which technique applies, which skips the coding overhead and drills recognition directly. That's the muscle that fails under interview pressure.
- **Decision flowcharts.** Question-based routing — is the input sorted, is the range contiguous, is it a tree or a graph — that leads you to a likely pattern when you're staring at a problem with no idea where to start.
- **A three-step structure per problem:** identify the pattern, implement it, then analyze time and space complexity and why.

Two things it doesn't do, and you should know before paying. There is **no video content at all** — it's text, diagrams, and an in-browser editor, so if you learn by watching someone solve problems, this will feel dry. And the problem bank is much smaller than LeetCode's, so most candidates still use it for the frameworks and drill volume elsewhere. It also isn't a system design or behavioral prep replacement; the coverage there is introductory.

Free practice lists are available with a free account if you want to test the format first — 👉 [try the AlgoMonster pattern problems before paying](https://algo.monster?fpr=li69).

## AlgoMonster plans and pricing

There are three tiers, and no monthly option. The paid plans bundle the pattern curriculum, unlimited Speedrun, real question sets from major tech companies updated monthly, an AI assistant, and the company's bonus courses (CS fundamentals and a build-an-LLM course) at no extra cost.

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited access to sample lessons and problems; free practice lists (Blind 75, Monster 50) with an account | $0 | — | [Start with the free tier](https://algo.monster?fpr=li69) |
| Pro — Yearly | All patterns, lessons and problems; unlimited Speedrun; unlimited AI assistant; company question sets; 12 months of full access; member discount on live bootcamps | $99/year as currently listed ($8.25/mo equivalent), with a 3-day free trial on the annual plan at the time of writing | Billed annually, renewable | [See the yearly plan](https://algo.monster?fpr=li69) |
| Pro — Lifetime | Everything in the annual plan, plus lifetime updates and an internal referral to a top tech company for qualified candidates | $189, billed once as currently listed | One-time payment | [See the lifetime plan](https://algo.monster?fpr=li69) |

A caveat that will save you confusion: AlgoMonster runs promotions close to continuously, and the headline numbers move with them. Different visits to the subscribe page show different prices — the annual plan has been listed at $69.99 and at $99, and lifetime at $89 and at $189, with the struck-through list prices running higher ($198/year and $378 for lifetime appear on partner pages for the current half-off sale). Third-party coupon listings also circulate codes that stack an extra 10% on top of the sale. Treat any single number you see quoted online, including the ones above, as a snapshot: 👉 [check the current AlgoMonster pricing and any live discount here](https://algo.monster?fpr=li69) before you buy.

### Who should pay, and who shouldn't

Worth it if your bottleneck is pattern recognition rather than coding speed, you have a compressed timeline, and you're fine reading instead of watching. The one-time lifetime option also makes sense if you expect to interview more than once — engineers who change jobs every two or three years get the math to work in their favor.

Skip it if you need volume. A few hundred problems teaches frameworks; it does not build the fluency that comes from solving hundreds more. Budget for a second resource for raw drilling, and for system design and behavioral prep if you're targeting senior roles. There's also no coaching, no mentorship, and no mock interviews with real engineers — it's entirely self-paced.

## The short version

Two pointers is one idea with three shapes. Converging pointers work because sorted order lets you discard an element permanently on every move. Fast-slow pointers work because the gap between the two indices carries information you'd otherwise have to store. Sliding windows are converging pointers' cousin, with a constraint you maintain incrementally.

The implementation is five lines. The interview is won or lost on recognizing which shape applies, stating why a discarded element can never be part of the answer, and getting the loop condition right.

Once you can do that out loud before writing code, you've learned the pattern — and 👉 [you can practice that exact skill set here](https://algo.monster?fpr=li69) instead of grinding problems one at a time and hoping the recognition shows up on its own.
