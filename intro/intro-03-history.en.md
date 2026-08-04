# A Short History of AI: The Rollercoaster, 1950 → Now

> 📅 2026-08-04 · Core Concepts
> AI was not born in the last five years — it has lived for seventy years and died twice. Understand the rhythm of springs, winters, and comebacks, and you will understand where today’s AI comes from.

---

AI did not pop into existence two or three years ago. It was "born" in 1950 and has officially "died" twice since — each time being hyped to the sky, then crashing into a winter.

But the point is not that it died. It is that **every spring left a foundation** — the lessons of the winters got picked up and relit in a new form. Understand that rhythm and you will see why "now" is different — that is the subject of `intro-04-why-now`. Here, we cover the seventy years first.

## The 1950s: where the dream began

Everything starts with two things arriving together: a machine that can compute, and a question about whether machines can think.

* **1950**: Turing publishes a paper on whether machines can think, proposing the famous **Turing test** — if a machine converses with you and you cannot tell it is a machine, it passes.
* **1956**: The Dartmouth workshop formally names "artificial intelligence," making AI a discipline. A program called the **Logic Theorist** proves mathematical theorems — often called the first AI program.
* **1957**: The **Perceptron** arrives — a very simple "learning machine." It is the ancestor of the "machine learning" idea from the layer cake in `intro-01`.

People back then were wildly optimistic. Some predicted that within twenty years machines would do everything humans can do. It did not happen.

## The 1970s: the first winter

Optimism hit a hard wall. In the 1970s researchers discovered that writing "intelligence" down as rules was a hundred times harder than imagined — a machine could play chess yet not follow one casual conversation. Even earlier, two Perceptron pioneers had published a book proving the Perceptron could not even learn XOR — knocking the simplest "learning machine" idea back to square one.

The UK's Lighthill Report was a downer, governments pulled funding, and money dried up. That was the **first AI winter**.

> Keep this lesson: between "sounds possible" and "actually works" sits a whole era of engineering. Every later winter is a reprint of that same sentence.

## The 1980s: expert systems, then the second winter

The 1980s warmed up again, on the back of **expert systems** — writing a doctor's diagnostic rules or a geologist's judgments into a program line by line. Businesses bought in, and Japan even launched a "Fifth Generation" computer project.

But they had a fatal flaw: the rules had to be hand-written, one by one — you could never finish, never be complete. Expectations fell short again, and in the early 1990s a **second winter** arrived. Ironically, the knowledge-representation and logic tools from that failed era became someone else's foundation.

## The 1990s–2000s: the quiet machine-learning revival

After two winters, the field learned its lesson: stop chasing "general intelligence" and instead do **one specific thing really, really well.**

* In 1997, IBM's **Deep Blue** beat chess world champion Kasparov — but via "brute-force search plus human-written chess tables," not by "learning to play," so few treated it as an "AI awakening."
* Spam filtering, credit-card fraud detection, handwriting recognition — these "narrow" tasks were handled beautifully with **statistics + machine learning**.
* Around 2010, "big data" made "more data is better" a mainstream belief for the first time.

These twenty years made no headlines, but they are the **silent foundation** of everything today. And it was during this stretch that the old idea of many-layered neural networks (deep learning) quietly revived in academic circles — just waiting for a spark.

## 2012: the night the fire started (AlexNet)

In 2012, a deep neural network called **AlexNet** crushed the runner-up by a wide margin at the ImageNet image-recognition competition. How? Not a cleverer algorithm — just **bigger, deeper, and trained on GPUs.**

The meaning: everyone suddenly realized **"deep" and "big" actually work.** Deep learning became mainstream, and 2012 is often called the "year zero" of modern AI. And the intuition that "bigger is better" later became a predictable math — the famous scaling curve in `train-05-scaling-laws`.

## The 2016 seed: AlphaGo

In 2016, DeepMind's **AlphaGo** beat Go world champion Lee Sedol. Go was considered the holy grail of human intelligence — combinatorial explosion so large that brute-force search was hopeless; only "intuition" worked. The moment AlphaGo won that match, the world firsthand saw "neural networks + self-play" do what humans could not.

It did not immediately set off language AI, but it proved the same thing: **deep learning was not just about recognizing cats.** A preview that it could do big things was planted.

## 2017: Transformers — scale finally found the right vehicle

Deep learning had taken off, but the star tasks were image ones that needed a label per image. What actually launched *language* was a 2017 paper: **"Attention Is All You Need," the Transformer architecture.**

* It parallelizes far better than the recurrent networks before it, so far more GPUs can train it at once.
* It is naturally built for "read a lot of text, guess the next word" — the exact trick from `llmcore-01-next-token`.
* It **welded scaling laws to language**, turning "make it bigger and it talks better" into a replicable engineering path.

## Seeing seventy years in one glance: a rollercoaster of expectations, a slope of capability

Compress all this history into one picture and you will see two completely different curves:

```text
expectations (swing wildly)     capability (rises steadily)
1956 dawn  ╱╲
╱  ╲______  1970s first winter
1980 thaw  ╱╲
╱  ╲______  1990s second winter
2000 narrow ML              ──── quiet accumulation (the foundation itself)
2012 onward  ╱╲____        2022 public spring, highest point yetThe key: every time expectations crashed, capability did not collapse with it
— the foundation was always kept
```

This picture is the key to the whole history: **technology climbs steadily while expectations crash up and down.** Right now you are standing at the highest point of the capability curve.

## 2022: ChatGPT — the whole world noticed

Between 2018 and 2021 the models were already getting stronger, but they were hidden inside APIs and papers. Then in **November 2022, ChatGPT launched**, dropping a conversational language model into everyone's browser.

* It used no revolutionary new algorithm — just Transformers plus the "obedience" taught by `train-02-finetuning` and `train-03-rlhf`.
* The real revolution was the **interface**: a chat box is the most natural interface there is, and everyone knows how to use it.
* It hit 100 million users in two months — the fastest consumer-app growth ever recorded.

If you ask "when did AI blow up," that is 2022. If you ask "when did AI get strong," that is a decade of slow accumulation. Those are two different things.

## Now: agents and multimodality

After 2024 the focus moved from "can chat" to "can do things" and "can see the world":

| What is happening now | Direction | In depth |
|---|---|---|
| **Agents**: plan, act, check results on their own | Making the model complete tasks, not just answer | `agents-01-what-is-agent` |
| **Multimodality**: understanding images, audio, video together | Expanding from "text" to "the world" | `models-03-multimodal` |
| **Reasoning**: thinking longer before answering | More reliable on math and logic | `models-04-reasoning` |

"Now" is still in progress. Whether this stretch is another spring to be written into history, nobody knows — but one thing is certain: this spring stands on the previous foundations, and it stands especially deep.

## A timeline, seventy years

| Year | Event | Meaning |
|---|---|---|
| 1950 | Turing test | A criterion for "can machines think" |
| 1956 | Dartmouth workshop | "Artificial intelligence" gets its name |
| 1970s | Optimism falls short | First winter |
| 1980s | Expert-systems boom | The seed of the second winter |
| 1990s–2000s | Narrow ML applications | The silent foundation |
| 2012 | AlexNet | Deep learning catches fire |
| 2016 | AlphaGo | The preview that DL can do "big things" |
| 2017 | Transformer | Language and scale get welded together |
| 2022 | ChatGPT | The world discovers AI |
| Now | Agents / multimodality | From "can chat" to "can do" |

## Each era’s "prediction vs. reality"

The most interesting part of history is looking back at what people expected to happen. This table is worth savoring:

| Era | The prediction then | What actually happened |
|---|---|---|
| 1950s | Machines will do everything humans do within twenty years | Could not even follow a casual conversation |
| 1980s | Expert systems will replace doctors and engineers | The rules could not be finished; the boom shattered |
| 1990s | AI is "dead", nobody talks about it | Narrow apps quietly ate a layer out of every industry |
| 2012 | "The era has truly arrived" | Yes — but only in academia, only in images |
| 2016 | "After AlphaGo comes AGI" | No — the next step was a better chatbot |
| 2022 | "Doom / miracle" theories flying in both directions | What arrived was a more useful, more error-prone narrow AI |

Every era mistakes "current progress" for "the arrival of the end." Exactly what `intro-05-agi` will warn you about.

## What this history tells you

**First, a winter is not "AI died" — it is "expectations fell."** The technology kept growing; it just could not keep up with the imagination.

**Second, every spring is built on the ashes of the last failure.** Today's Transformer uses knowledge-representation concepts from 1980s research, the statistical tools of 2000s machine learning, and the scale intuition of the 2010s.

**Third, and most practically:** when you hear "AI will change the world again" or "AI is dead," ask yourself "is this about expectations, or capabilities?" Capabilities climb steadily; expectations swing wildly. Hold that thread and the headlines will stop steering you.

#### Want more detail on a few eras? Open here

Deep Blue 1997: won by “brute-forcing faster,” not by learning, which is why it was never treated as evidence of general intelligence.\n\nWhy AlexNet is the “year zero”: it proved the “deep + big + GPU” trio really crushes traditional methods, and from then on deep learning got all the resources.\n\nAlphaGo self-play: instead of human game records it played itself millions of games — a stunning demonstration of learning from zero to the top, nothing like memorizing games.

> The one sentence to remember: AI history is not the linear growth of technology — it is a gentle slope of capability riding a steep rollercoaster of expectations. Today we stand on the slope at its highest point. Let expectations keep swinging.

## Next

The history is covered, so the natural question follows: **then why "now"?** People have hyped AI for seventy years — why did the last five genuinely explode? The answer is in `intro-04-why-now`: the quartet of scaling laws, data, capital, and interface.

#### Q: AI history had two “winters.” What is the most accurate way to understand them?

* AI research fully stopped for twenty years and nobody worked on it until 2012

* Funding and enthusiasm receded and expectations fell, but underlying research kept accumulating, leaving foundations for the next spring

* The winters happened because governments passed laws banning AI research

* The winters never really existed — the term is just media exaggeration

> 💡 A winter is about falling expectations and dried-up funding, not dead research. The knowledge-representation work of the 1980s and the statistical ML of the 2000s are exactly the foundations laid during winters and picked up later.
