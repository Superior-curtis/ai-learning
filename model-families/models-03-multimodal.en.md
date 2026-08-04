# Multimodal Models: When Models Open Their Eyes

> 📅 2026-08-04 · Core Concepts
> Models no longer read only text: they see images, hear audio, watch video. A look at how a vision encoder + LLM combine, and when multimodal truly matters — OCR, screenshots, diagrams, accessibility.

---

You can finally drop an indecipherable error screenshot into a chat box and ask: "what does this mean?" It reads it. That is **multimodality** — models no longer eat only text; they open their eyes and prick up their ears.

Last stop (`models-02-families-tour`) we toured the model families. This post dives into one ability that changed our lives the most: how models started to "see" and "hear." Let me make clear that multimodal isn't magic — it's two parts bolted together. By the end you'll know **when multimodality is genuinely worth using**, and **when it will quietly lie to you.**

## Back to the one rule

The foundation of this whole book (`llmcore-01-next-token`) is: a model basically just **predicts the next token**. Sounds like text only — right? Yes, if text is all it ever sees.

Multimodal is exactly the twist: **it translates images, sound, and video into "the language of tokens."** The model doesn't become some other creature; it just discovers "oh, an image can also be translated into a string of tokens I understand," then keeps using its only trick — guessing one token at a time.

That doesn't make multimodal unimportant. Quite the opposite: **the ability to translate the world into tokens is exactly what lets a model step out of its text-only cage.** A huge share of information in our daily workflows isn't in text at all — it lives in screenshots, photos, meeting recordings, and drawn diagrams. Multimodality is the bridge that translates that back into the world of words.

## How an image "enters" the model

Let's unpack an image. An image is essentially a mass of numbers (each pixel's color). To read it, the model can't just swallow those numbers — it needs an interpreter: a **vision encoder**.

The two-stage design is the shared skeleton of modern multimodality, broken into four steps:

#### Chunk

Cut the image into fixed-size patches, like puzzle pieces.

#### Encode

Each patch is turned into a vector by the vision encoder, capturing the visual info of that region.

#### Align

A projection maps the visual vectors into the text-token space so the two can be read together.

#### Fuse

Vision tokens and text tokens go into the LLM together, which keeps predicting one token at a time.

Key idea: **the vision encoder does the "seeing"; the LLM does the "talking."** Each does its own job, and together they form a complete multimodal model. There is no "new intelligence" — only "new input plugs."

| Part | Its job | Output |
|---|---|---|
| Vision encoder | Turns image patches into vectors | Vision tokens |
| Projection layer | Aligns vision vectors to the text-token space | Mixed token sequence |
| LLM backbone | Fuses and responds via the next-token mechanism | Text / other output |

## Audio and video: the same recipe

Images were the first unlocked input pipe, but by no means the last. **Audio and video follow the identical logic: first encode the non-text content into tokens, then hand it to the same LLM.**

* **Audio**: speech gets transcribed into text (speech recognition), or an audio encoder turns it directly into tokens. Meeting notes and voice memos become reading material for the model.
* **Video**: a video is really a fast sequence of images. The model extracts key frames, encodes each visually, and uses their order to understand "what happens next."

This has very practical meaning for us: **almost any "non-text" material you have today can become model input.** No more hand-typing audio into transcripts or describing images into words — just hand the raw material over.

## What multimodal is actually good for

Multimodality isn't there to "appreciate pictures" — it's there so models can handle **inputs we naturally have in the real world**. A few moments that actually show up in your work:

* **OCR and documents**: scan a PDF or snap a photo, then ask "what conditions does page three of this contract state?" The model reads the text in the image and understands context around it.
* **Screenshot debugging**: paste an error screenshot and ask "what does this error mean" — no typing required, the model reads it directly.
* **Charts and diagrams**: drop in an architecture or flowchart and ask it to explain "how this system works." It reads the layout, not just the text.
* **Accessibility**: turning a photo into a text description so a visually impaired user can "see" it too. A real, valuable application.

What these have in common: **the information lives in the image; a text-only model can't reach it, and multimodal opens that door.** Pair it with RAG (`llm-03-rag`) and you can build systems that "answer questions over a whole pile of document screenshots."

Take a terminal agent like Claude Code as an example: it reads an image the moment you paste a screenshot, turning your debugging scene into its input (the `claude-code` series is full of such flows). This isn't "one more feature" — it's a change in how you work. **You and the model start sharing the same screen.**

## A minimal multimodal call

Most multimodal models accept an image through their API. In structure it's just "one more image field in the conversation," everything else unchanged:

```python&#x20;—&#x20;answer&#x20;with&#x20;an&#x20;image
# pseudocode: the shape, each SDK varies slightly
messages = [
{ "role": "user", "content": [
    { "type": "image", "image": screenshot_bytes },
    { "type": "text", "text": "What is this error?" },
]}
]
reply = client.chat(messages)
print(reply)
```

Two details to note: first, the **image sits in the content array alongside the text**, and the model reads both together; second, **don't lazily throw a full-size image in** — downscale or crop locally first, and you'll save a surprising amount of tokens and latency.

## The cost of multimodal: tokens aren't cheap

Translating an image into tokens has a price. A single image can swallow **hundreds to thousands of tokens** — far more than "putting one sentence into words." Several practical consequences follow:

| Usage | Rough cost | Advice |
|---|---|---|
| One text question | Tens of tokens | Cheapest |
| One small image + a sentence | Hundreds of tokens | Common, fine |
| One dense document page | Thousands of tokens | Skimp where you can; crop before sending |
| One long video | Very expensive | Only pull key frames |

So one practical habit of multimodality: **when you can downscale, crop, or extract frames first, don't throw in the whole thing.** Every token you save is real money and waiting time.

> Remember one thing: multimodal doesn't make a model "smarter," it makes it "able to see." The engine underneath is still that next-token predictor, just now able to translate images, sound, and video into tokens. The model's intelligence didn't change — its input bandwidth grew, and bandwidth costs money.

## Three levels of multimodal, don't mix them up

Not every model that "handles images" is the same. There are three levels; understanding them avoids confusion:

| Level | What it does | Example |
|---|---|---|
| **Pure vision understanding** | Turns a chart into a description | captioning models |
| **Vision + text understanding** | Takes image + text in, outputs text | reading screenshots, answering image questions |
| **Generation + understanding** | Both reads and can draw images | alternating image-to-text and text-to-image |

Most "multimodal LLMs" sit in the middle layer: **read an image in, produce text out.** It can "understand" an image and answer you in language — which is already plenty for most of our work. Models that both read and generate are stronger, but pricier and harder to control.

## It also has real limits

Let's honestly admit multimodal isn't omniscient. Your own usage will confirm these boundaries:

* **It reads "the translated image," not the original.** Small text, dense tables, and corner details of an image often get lost. Don't ask it to "quote the contract line by line" — it will make things up.
* **It still hallucinates.** The model reading an image is as prone to confidently being wrong as when reading text (`llmcore-05-hallucination`). Image input doesn't make it honest.
* **Vision isn't "really seeing."** It has no eyes; it only has a special way of tokenizing. It "knows" statistical patterns of images, not human sight.
* **Multi-step visual reasoning is still shaky.** Ask "how many red round objects are in this image" and it may miscount — counting and spatial relations are a vision model's weak spot.

## What to watch for when choosing a multimodal model

Not every "multimodal" model is equally reliable. When buying or comparing, a few things to observe:

| Dimension | The question to ask |
|---|---|
| **Resolution sensitivity** | How small can it read? Dense tables? |
| **Grounding ability** | Can it point to "which corner of the image the problem is in" |
| **Multi-image reasoning** | Can it compare two images and find the difference |
| **Consistency** | Ask the same image three times — is the answer stable |

Test it against your own real screenshots and documents (the `models-06-evaluation` method) rather than trusting official demos — those tend to show only the images the model is good at.

## So what multimodal means for you

In one phrase: **it gives your workflow a new input channel.** Now, when you meet an image or audio, you no longer need to manually transcribe it into text before asking — just throw the raw material at the model.

In the world of multimodal models, the most practical rule is: **"if there's an image, don't paste a text description; give the image."** Let the vision encoder do what it's good at. Use your own task set (the `models-06-evaluation` approach) to measure how each multimodal model handles your images — more honest than any vendor claim.

Next, from "able to see" to "thinking deeper" — **reasoning models**: the ones that think for a while before they answer (`models-04-reasoning`).

#### Q: Why can a multimodal model read an image when a traditional text model can't? What's the key difference?

* It has an add-on feature that can draw

* It downloads image content from the internet

* The vision encoder turns the image into vision tokens, then the LLM processes them with the next-token mechanism

* It has ten times more parameters than a text model

> 💡 The key to multimodality is that the vision encoder translates the image into vision tokens, and the model then uses its single trick — predicting the next token — to fold image understanding into its text response. The intelligence didn't change; the input bandwidth grew.
