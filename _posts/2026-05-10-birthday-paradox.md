---
title: "Birthday Paradox"
date: 2026-05-10
---

Have you ever met someone who shares your exact birthday? It’s a peculiar feeling, isn’t it? Like running into a long-lost twin in a sea of strangers. We intuitively understand that the more people we know, the higher the chance of this happening. But our intuition often fails us when we shift the question slightly. What if we don’t care about matching _our own_ birthday, but simply ask if _any two_ people in a room share a birthday? The answer defies common sense and reveals a profound truth about probability that governs everything from cryptography to the randomness of life.

Consider this: to have a greater than 50% chance of finding someone who shares your specific birthday, you would indeed need to know around 253 people. The math is straightforward. The probability that one person does _not_ share your birthday is $364/365$. For $k$ people, the probability that _none_ of them share your birthday is $(364/365)^k$. Therefore, the probability of at least one match is the inverse: $1 - (364/365)^k$. When $k = 253$, this probability ticks over to approximately 50.04%.

However, if we remove the ego from the equation and ask a broader question—"Do any two people in this group share a birthday?"—the numbers collapse dramatically. We are no longer looking for a specific match; we are looking for any collision. For a group of $k$ people, the probability that all birthdays are unique is calculated by considering each person in sequence. The first person can have any birthday. The second person must avoid that day (364 options), the third person must avoid the first two (363 options), and so on. The number of possible scenarios where all $k$ people have different birthdays is $365 \times 364 \times ... \times (365-k+1) = \frac{365!}{(365-k)!}$, while the total number of all possible birthday combinations for $k$ people is $365^k$. Therefore, the probability that all $k$ people have distinct birthdays is $P(\text{all distinct})=\frac{365!}{365^k(365-k)!}$. Thus, the probability of at least one collision is $1$ minus that value, $$P(k_s)=1-\frac{365!}{365^k(365-k)!}$$.

The result is startling. With just 23 people, the probability of a shared birthday exceeds 50%. In a crowded room of 60, it climbs to over 99%. This is the famous "Birthday Paradox," though there is no logical contradiction—only a collision between human intuition and statistical reality.

But let us push this further. What if we want to know the probability of _three_ people sharing the same birthday? This question takes us beyond the standard paradox and into the realm of occupancy problems attributed to Richard von Mises. [This article](https://0xkrt26.github.io/math_behind_security/2026/05/08/birthday-problem.html) provides a formula to calculate the expected value for the probability of at least $s$ people sharing a birthday among $k$ individuals:
$$E(k_s) = n \cdot \binom{k}{s} \cdot \left(\frac{1}{n}\right)^s \cdot \left(1 - \frac{1}{n}\right)^{k-s}$$
For $k=60$ and $s=3$, this suggests an expected value of $0.2197$, implying that in a group of $60$, we might expect a triple birthday to occur about once every five gatherings. However, while this figure is close to the correct value of $0.2072$, the calculation method itself is flawed. If we substitute $k=23$ and $s=2$, we get $E(23_2) = 365 \times \binom{23}{2} \times \left(\frac{1}{365}\right)^2 \times \left(\frac{364}{365}\right)^{23-2} = 0.6543$, which is far from the known probability of $0.5073$. If we take $k=30$, we get $E(30_2) = 1.1037$, a value that exceeds $1$.

Now, let's derive the correct calculation. Let's revisit the problem of at least two people sharing a birthday, but approach it from a different angle. This time, instead of considering the available days for each person one by one, we will distribute the $k$ people into the 365 days. Since we are calculating the scenario where everyone has a unique birthday, the number of days on which someone has a birthday must be exactly $k$ (if any two people shared a birthday, the number of occupied days would be less than $k$).

The number of ways to choose $k$ days out of 365 is $\binom{365}{k}$. Then, assigning these $k$ specific days to the $k$ people corresponds to calculating the number of permutations, which is $k!$. Therefore, the total number of possible scenarios where all birthdays are distinct is the product of these two values, $P(\text{all distinct})=\binom{365}{k}\times k!$.

Similarly, to calculate the probability of at least three people sharing a birthday, we first compute the probability that no day has more than two birthdays. Since the number of people celebrating on any given day can only be 2, 1, or 0, we can assume there are $j$ days where exactly two people share a birthday. Consequently, the number of days with exactly one birthday is $k-2j$.

We choose $j$ days out of 365 for the shared birthdays, which can be done in $\binom{365}{j}$ ways. From the remaining $365-j$ days, we then select $k-2j$ days for the single birthdays, which gives us $\binom{365-j}{k-2j}$ combinations.

Next, we select $2j$ people out of the $k$ individuals to form the pairs, which can be done in $\binom{k}{2j}$ ways. We then assign these $2j$ people to the $j$ selected days (2 people per day). The number of ways to distribute them is $\binom{2j}{2}\binom{2j-2}{2}...\binom{2}{2}=\frac{(2j)!}{2^j}$. Finally, we arrange the remaining $k-2j$ people into the $k-2j$ single-birthday days, which can be done in $(k-2j)!$ ways.

Multiplying these terms together, we get the number of scenarios where exactly $j$ pairs share a birthday and the remaining $k-2j$ people have unique birthdays: $Count(k_2,j)=\binom{365}{j} \times \binom{365-j}{k-2j} \times \binom{k}{2j} \times \frac{(2j)!}{2^j} \times (k-2j)!$.

Since $j$ can range from 0 to $\left\lfloor \frac{k}{2}\right\rfloor$, we sum all these scenarios to find the total number of cases where at most two people share a birthday: $\sum _{j=0}^{\frac{k}{2}} Count(k_2,j)$.

Thus, the probability that at least three people in a group of $k$ share the same birthday is:
$$P(k_3)=1-\frac{\sum _{j=0}^{\frac{k}{2}} Count(k_2,j)}{365^k}=1-\sum _{j=0}^{\frac{k}{2}} \frac{\binom{365}{j} \times \binom{365-j}{k-2j} \times \binom{k}{2j} \times (2j)! \times (k-2j)!}{365^k \times 2^j}$$

To validate our calculations, we will introduce a random simulation.
<iframe width='800' height='400' src='https://www.wolframcloud.com/obj/0c6fa951-d1c3-4776-a18e-1cb944167b11' frameborder='0'></iframe>

This principle transcends parties and social gatherings. In computer science, it is known as the "Birthday Attack." Cryptographic hash functions act like digital birthdays, mapping data to a fixed set of outputs. An attacker does not need to find a specific hash collision; they merely need _any_ two inputs that hash to the same value. Because of the math we just explored, this can be achieved in roughly the square root of the time required for a brute-force search. It is a testament to the power of combinatorics—a power that turns the seemingly impossible into the statistically probable.

The mathematics above assumes a uniform distribution of birthdays. But in reality, births are not random. Seasonal variations, cultural habits (like holiday conceptions), and medical interventions (like scheduled C-sections) create peaks and troughs in the birth calendar. If the distribution of birthdays is not uniform, does the probability of a collision become higher or lower? The answer lies in the concept of entropy. A uniform distribution represents maximum randomness, or maximum dispersion. When we introduce skew—clustering births around certain dates—we reduce the "effective" number of days in the year. It is as if the 365 days contract into a smaller pool of high-probability dates. Consequently, collisions become _more_ likely. Non-uniformity favors the matchmaker. In a world where everyone tries to be unique, coincidences are rare; but in a world where we follow trends and seasons, collisions are inevitable.

This is also why a robust Pseudo-Random Number Generator (PRNG) is crucial. If the distribution of the generated random numbers is not sufficiently uniform, it will inevitably increase the probability of collisions.

So, the next time you are in a room with 22 strangers, make a bet. The odds are better than even that two of you share a birthday. It is not magic; it is the beautiful, inevitable machinery of probability grinding the odds in your favor.
