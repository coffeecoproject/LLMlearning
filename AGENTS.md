# LLMLearning workspace guidance

This directory is both an Obsidian vault and a Codex workspace.

## Purpose

Help a non-specialist user understand the path from large language models to reasoning, agents, continual learning, self-improving systems, embodied intelligence, and AGI. Large language models remain the first and foundational module.

## Current learning phase

- Follow the seven-module learning structure defined in `00-总纲/00-AI 智能系统学习总纲.md`. The current content-development phase starts with large language models before later modules are expanded.
- Explain LLMs thoroughly for a non-specialist audience. Do not assume machine-learning, advanced mathematics, or neural-network background.
- Use the learning-depth model below. A topic may be complete at the framework level without forcing the reader through parameters, mathematics, implementation, or architecture variants.
- Never trade accuracy for a simple analogy. Clearly label where an analogy stops matching the real mechanism.
- Define every new technical term on first use and avoid unexplained acronym chains.

## Learning depth and reading paths

Organize substantial topics as progressive reading layers. These are curriculum layers, not components of the model itself.

### Level 0: framework overview

- Serve readers who only want to understand the overall LLM architecture.
- Answer only the essential questions: where the concept is, what enters it, what it does, why it is needed, what it outputs, and what it is not.
- Use one compact flow and, when helpful, one concrete example.
- Do not require formulas, tensor shapes, parameter counts, model configuration fields, research papers, or architecture variants.
- End with a clear stopping point: a reader who can explain the framework may continue to the next major component without opening deeper notes.

### Level 1: basic mechanism

- Explain the causal process step by step and connect it to upstream and downstream components.
- Define the key terms needed to reconstruct the mechanism rather than memorize a slogan.
- Use a small worked example when it materially improves understanding.
- Distinguish the concept from its nearest neighboring concepts and explain what would fail without it.

### Level 2: parameters and technical depth

- Cover tensor shapes, learned parameters, configuration fields, parameter scale, and simple numerical calculations.
- Keep mathematics optional and use small numbers before symbolic notation.
- Do not make this layer a prerequisite for readers who only need the framework or basic mechanism.

### Level 3: variants, evidence, and implementation boundaries

- Cover modern architecture variants, open-model configurations, research interpretations, and relevant training or runtime implementation details.
- Use primary sources and separate published facts, inference, and unknown information.
- Keep model internals distinct from inference runtimes, serving systems, Agent frameworks, and product behavior.

### Applying the layers

- Every major topic overview must offer explicit routes such as `只看框架`, `理解机制`, and `继续深入`.
- Never label all depth layers as required reading. Required reading must be relative to a stated learning goal.
- Judge completeness within a layer: framework completeness does not require mathematical or implementation completeness.
- Not every topic needs four physical subfolders. Create a layer folder only when it owns several notes or is expected to grow; otherwise express the layers as sections and links in the topic overview.
- Do not repeat the full explanation at every layer. Give each concept one canonical detailed note; higher-level notes summarize and link to it.

## Note conventions

- Write learning notes primarily in Chinese; retain standard English technical terms where useful.
- Use Obsidian `[[Wikilinks]]` to connect LLM concepts, mechanisms, experiments, and misconceptions.
- Prefer small, focused concept notes over long unstructured transcripts.
- Match each note to one learning layer and one note role. Do not force every note to contain every teaching element.
- Across a complete topic, normally provide: a one-sentence understanding, why the concept exists, a concrete example, the real mechanism, adjacent relationships, common misconceptions, and a short comprehension check. Distribute these across the appropriate notes instead of repeating them in every note.
- Separate framework reading, mechanism reading, and optional mathematical or implementation details visibly in both the topic overview and the file tree when separate folders are justified.
- Atomic means one core question per note, not shallow coverage. Explain that question deeply enough that the user can reconstruct the causal chain without memorizing slogans.
- For substantive concepts, cover prerequisite connection, intuitive entry, step-by-step mechanism, worked example, adjacent relationships, boundary cases, common misconceptions, optional technical detail, and comprehension exercises across the topic as applicable. Do not turn this list into a mandatory template for every individual file.
- Prefer causal explanations in prose over lists of facts. Explicitly answer “why this step is necessary” and “what would break without it.”
- A framework note is complete when the reader can place the concept in the system and explain its input, role, output, and boundary. A mechanism topic is complete when the reader can also distinguish it from neighboring concepts and predict its behavior in a new example.
- Overview notes own the map, scope, learning routes, and progress. They should not duplicate the full content of their child notes.
- Framework notes should remain compact and avoid introducing terms that are only needed by deeper layers.
- Mechanism notes should prioritize causal explanation and one coherent example over many disconnected facts.
- Parameter or mathematical notes must be visibly optional and must explain what each calculation helps the reader understand.
- Reference notes may focus on evidence and source boundaries; review notes may focus on misconceptions and comprehension checks. They do not need to imitate the structure of a mechanism lesson.
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
- Design the tree for Obsidian's native alphabetical, folders-first explorer. Do not rely on note creation time, manual drag order, or an unapproved community plugin to express the curriculum sequence.
- Keep `学习主页.md` at the vault root as the intentional global-entry exception. Curriculum notes inside module and topic folders must follow the numbering rules below.
- Prefix ordered sibling folders with a two-digit local order, such as `01-LLM`, `01-基础结构与计算机制`, `01-Tokenizer文本离散化系统`, and `02-Embedding向量表示`. Each number describes order only among the items in that directory, not complete ancestry.
- Name a topic folder after the highest-level object, mechanism, or system being studied, followed by its role when clarification is useful. Prefer `Tokenizer文本离散化系统` over a loose collection name such as `文本离散化与Token`.
- When a directory contains both child folders and an overview note, put the overview in the first child folder: `00-概览/00-<主题>概览.md`. A loose Markdown file beside child folders will appear after all folders in Obsidian and must not be used as the visual entry point.
- Give each topic one semantically named entry note ending in `概览`, such as `00-概览/00-Tokenizer文本离散化系统概览.md`. The topic folder expresses hierarchy; the overview note owns the topic map, reading order, progress, concept boundaries, and links to subsection notes.
- Do not use ancestry-encoded dotted numbering such as `1.1`, `1.1.1`, or `1.4.2.4.3` in folder names, filenames, headings, Wikilink labels, or frontmatter section fields.
- Represent each stable subdivision that owns several notes as a locally numbered subfolder. In a leaf folder, number the overview `00` and number the remaining notes in reading order, for example `01-Token/00-Token概览.md`, `01-Token/01-为什么模型需要Token.md`, and `01-Token/02-Token到底是什么.md`.
- When depth layers need their own folders, use clear local names such as `01-框架速览`, `02-基础机制`, `03-参数与深入`, and `04-扩展结构`. These folder names describe the reading depth, while the topic overview must still describe the technically accurate system structure.
- Prefix every note in an ordered learning directory with a two-digit local sequence. Use `00` for that directory's overview and `01`, `02`, and so on for its declared reading order. Name the semantic portion after one core concept or one core question, such as `01-为什么模型需要Token.md`.
- Derive note numbering from the explicit reading sequence in the parent overview. Alphabetical order must display the same sequence as the overview links; mixed Chinese and English titles must not be left to sort themselves.
- Keep numeric prefixes out of rendered prose when they reduce readability. Prefer an alias such as `[[01-为什么模型需要Token|为什么模型需要 Token]]` while keeping the numbered target filename.
- Do not add a numeric prefix to a folder merely because it is nested. Add one only when sibling order conveys a curriculum or mechanism sequence; otherwise use the semantic name directly.
- Do not create a filesystem level for every minor conceptual distinction. Create a subfolder only for a stable subsection that owns multiple concepts or is expected to grow; express smaller relationships in its entry note with `[[Wikilinks]]`.
- Classify concepts by their actual role before placing them in an outline. In particular, distinguish: object, resource, output, representation route, algorithm, implementation tool, and downstream effect. Do not present neighboring categories as parent-child relationships. For example, Token is an object; Vocabulary is a resource; Token ID is an output representation; BPE, WordPiece, and Unigram are methods; tiktoken and SentencePiece are tools or implementations.
- Distinguish system structure from learning order. The system outline should be technically accurate, while the reading sequence should progress from observable input/output and concrete examples toward internal mechanisms.
- When inserting a note into an existing reading sequence, renumber the affected sibling notes rather than appending a misleading number. Update the parent overview's reading order in the same change.
- When renaming, renumbering, or moving a note or folder, update all Wikilink targets and aliases, explicit paths, frontmatter relationships, topic outlines, module outlines, and `学习主页.md`; then check the entire vault for stale names, broken links, and duplicate basenames.

## Teaching examples and mathematical depth

- Use small, concrete examples throughout the required reading, including Chinese text, English text, numbers, code, and Emoji when they reveal different model behavior.
- Show the observable transformation first, then explain why it occurs and what mechanism produces it.
- Do not add every example type to every note. Choose the smallest example that reveals the behavior under discussion.
- Do not make mathematical formulas a prerequisite for framework or basic-mechanism reading. Put formulas, derivations, parameter calculations, and implementation-level details in a clearly labeled optional note or section.
- Introduce a mathematical idea with small concrete numbers before symbolic notation. Include a calculation only when its result clarifies a structural relationship.
- A simplified example must be labeled as illustrative when its tokens or IDs are invented. Never imply that illustrative Token IDs are values from a real model.

## Safety

- Do not install Obsidian community plugins without explicit user approval.
- Preserve user-authored notes and unrelated workspace changes.
