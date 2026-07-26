# LLMLearning workspace guidance

This directory is both an Obsidian vault and a Codex workspace.

## Purpose

Help a non-specialist user understand the path from large language models to reasoning, agents, continual learning, self-improving systems, embodied intelligence, and AGI. Large language models remain the first and foundational module.

## Current learning phase

- Follow the seven-module learning structure defined in `00-总纲/AI 智能系统学习总纲.md`. The current content-development phase starts with large language models before later modules are expanded.
- Explain LLMs thoroughly for a non-specialist audience. Do not assume machine-learning, advanced mathematics, or neural-network background.
- Prefer progressive depth: intuition first, then mechanism, then an accurate technical model, with mathematics in an optional section.
- Never trade accuracy for a simple analogy. Clearly label where an analogy stops matching the real mechanism.
- Define every new technical term on first use and avoid unexplained acronym chains.

## Note conventions

- Write learning notes primarily in Chinese; retain standard English technical terms where useful.
- Use Obsidian `[[Wikilinks]]` to connect LLM concepts, mechanisms, experiments, and misconceptions.
- Prefer small, focused concept notes over long unstructured transcripts.
- Each teaching note should normally include: one-sentence understanding, why the concept exists, a concrete example, the real mechanism, common misconceptions, and a short comprehension check.
- Separate essential reading from optional mathematical or implementation details.
- Atomic means one core question per note, not shallow coverage. Explain that question deeply enough that the user can reconstruct the causal chain without memorizing slogans.
- For substantive concepts, cover these layers where applicable: prerequisite connection, intuitive entry, step-by-step mechanism, worked example, relationship to adjacent concepts, boundary cases, common misconceptions, optional technical detail, and comprehension exercises.
- Prefer causal explanations in prose over lists of facts. Explicitly answer “why this step is necessary” and “what would break without it.”
- A concept is not complete merely because its definition was given. Continue until the user can distinguish it from neighboring concepts and predict its behavior in a new example.
- When suitable, add an `开放模型观察` section using a precisely named open-weight model version.
- Use primary sources for model settings: official model cards, official repositories, checked-in configs, or technical reports.
- Separate published facts from inference and unknown information; record the source and verification date for version-sensitive settings.
- Include OpenAI models in the observation set, but match the claim to the available evidence. Use OpenAI open-weight models such as a precisely named `gpt-oss` version for inspectable architecture and configuration examples. Use closed GPT models only for officially documented interfaces, runtime behavior, and capability observations; mark unpublished internal architecture as unknown and never infer it from product behavior.
- When turning a conversation into notes, distinguish facts, the user's current understanding, open questions, and conclusions from experiments.
- When one topic spans distinct phases, label them explicitly as `训练阶段`, `运行阶段`, and where needed `两阶段共同`. Do not interleave tokenizer training, LLM training, model inference, and generation decoding without first stating which phase and system layer is being discussed.
- Keep the root learning map `学习主页.md` updated when adding a major topic.
- Store attachments under `99-Assets/`.
- Do not edit `.obsidian/` settings unless the user asks for an Obsidian configuration change.

## Knowledge structure and naming

- The Obsidian file tree must reflect the visible knowledge hierarchy. Do not change only a note heading when the agreed concept name or hierarchy has changed; rename the corresponding folder or file when appropriate.
- Organize content from broad to narrow: learning module → mechanism section → topic system → stable subsection → atomic concept note.
- Prefix only folders whose visible order matters with a two-digit local order, such as `01-LLM`, `01-基础结构与计算机制`, `01-Tokenizer文本离散化系统`, and `02-Embedding向量表示`. Each folder number describes its order among its siblings, not its complete ancestry.
- Name a topic folder after the highest-level object, mechanism, or system being studied, followed by its role when clarification is useful. Prefer `Tokenizer文本离散化系统` over a loose collection name such as `文本离散化与Token`.
- Give each topic folder one semantically named entry note ending in `概览`, such as `Tokenizer文本离散化系统概览.md`. The folder expresses expandable hierarchy; the clearly named overview note owns the topic map, reading order, progress, concept boundaries, and links to subsection notes.
- Do not use ancestry-encoded dotted numbering such as `1.1`, `1.1.1`, or `1.4.2.4.3` in folder names, filenames, headings, Wikilink labels, or frontmatter section fields.
- Represent each stable subdivision that owns several notes as a locally numbered subfolder with a semantic overview note, such as `01-Token/Token概览.md` and `02-Vocabulary与Token ID/Vocabulary与Token ID概览.md`.
- Name atomic notes directly after one core concept or one core question, such as `为什么模型需要Token.md`. Keep their reading sequence in the parent overview note instead of encoding the entire sequence in every filename.
- Do not add a numeric prefix to a folder merely because it is nested. Add one only when sibling order conveys a curriculum or mechanism sequence; otherwise use the semantic name directly.
- Do not create a filesystem level for every minor conceptual distinction. Create a subfolder only for a stable subsection that owns multiple concepts or is expected to grow; express smaller relationships in its entry note with `[[Wikilinks]]`.
- Classify concepts by their actual role before placing them in an outline. In particular, distinguish: object, resource, output, representation route, algorithm, implementation tool, and downstream effect. Do not present neighboring categories as parent-child relationships. For example, Token is an object; Vocabulary is a resource; Token ID is an output representation; BPE, WordPiece, and Unigram are methods; tiktoken and SentencePiece are tools or implementations.
- Distinguish system structure from learning order. The system outline should be technically accurate, while the reading sequence should progress from observable input/output and concrete examples toward internal mechanisms.
- When renaming or moving a note or folder, update all explicit paths, frontmatter relationships, topic outlines, module outlines, and `学习主页.md`; then search the vault for stale names or broken path references.

## Teaching examples and mathematical depth

- Use small, concrete examples throughout the required reading, including Chinese text, English text, numbers, code, and Emoji when they reveal different model behavior.
- Show the observable transformation first, then explain why it occurs and what mechanism produces it.
- Do not make mathematical formulas a prerequisite for the main learning path. Put formulas, derivations, and implementation-level calculations in a clearly labeled optional section.
- A simplified example must be labeled as illustrative when its tokens or IDs are invented. Never imply that illustrative Token IDs are values from a real model.

## Safety

- Do not install Obsidian community plugins without explicit user approval.
- Preserve user-authored notes and unrelated workspace changes.
