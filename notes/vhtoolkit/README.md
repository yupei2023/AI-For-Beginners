# Virtual Human Toolkit (VHToolkit) – Learning Notes

Project site: <https://vhtoolkit.ict.usc.edu/>
Maker: USC Institute for Creative Technologies (ICT)

> These notes are a personal study log. They are meant to move into a dedicated
> `vhtoolkit-learning-notes` repository once that repo exists.

## 1. What it is

The ICT Virtual Human Toolkit is a collection of modules, tools and libraries
for building **conversational virtual characters** ("virtual humans"): embodied
agents that can listen to a user, decide what to say, speak, and animate
appropriate non-verbal behaviour (gaze, gestures, head nods, lip sync).

Two audiences are called out in the official docs:

| Audience   | What they do                                            |
|------------|---------------------------------------------------------|
| Users      | Run the shipped components as-is, author new content    |
| Developers | Extend or replace modules; need programming + distributed-systems background |

## 2. Architecture overview

The toolkit is a **distributed, message-based** system. Independent processes
talk over a message bus rather than being linked into one binary. That makes
it easy to swap a module (e.g. a different speech recognizer) without touching
the rest.

```
 user speech/text
       |
       v
 AcquireSpeech / PocketSphinx  (speech & text input)
       |
       v   (VHMsg)
    NPCEditor   -- picks the character's reply line
       |
       v   (VHMsg)
      NVBG      -- plans non-verbal behaviour, emits BML
       |
       v   (BML over VHMsg)
    SmartBody   -- animates the character (gaze, gesture, lip sync, locomotion)
       |
       v
 vhtoolkitUnity -- Unity-based renderer + authoring/debug tools

 MultiSense (perception: face/head/gaze sensing) --> SmartBody / other modules
```

## 3. Core components

### VHMsg (messaging layer)
- Custom pub/sub messaging built on **Apache ActiveMQ**.
- Messages are usually broadcast; every module subscribes to what it needs.
- Client libraries exist for several languages (C++, C#, Java, Python...).
- *Takeaway:* the whole toolkit is glued together with this bus. Learn the
  message names first; debugging is mostly watching the bus.

### NPCEditor (dialogue)
- Authoring tool plus runtime for **question → answer** dialogue.
- Uses a **statistical text classifier**: user utterances are mapped to the
  closest authored response line. No full planning agent; it handles all
  verbal input/output.
- Content is authored as pairs of "questions people might ask" and "lines the
  character can say", then the classifier is trained.

### NVBG – NonVerbal Behavior Generator
- **Rule-based** behaviour planner.
- Reads the text the character is about to say, infers communicative functions
  (emphasis, affirmation, negation, etc.) and outputs a **BML** (Behavior
  Markup Language) schedule of gestures, head nods and gaze shifts.

### SmartBody (animation)
- Controller-based real-time character animation engine driven by **BML**.
- Handles locomotion, gaze, lip sync, reaching, and blending of behaviours.
- Can be used standalone outside the toolkit.

### MultiSense (perception)
- Framework for running several sensing modules at once (face tracking, head
  pose, gaze, etc.).
- Included in the toolkit as a proof of concept; publishes perception data
  (Perception Markup Language, PML, in older docs) and talks to SmartBody.

### vhtoolkitUnity (rendering)
- Unity-based renderer with authoring and debugging tools.
- Older releases referenced plain "Unity"; newer pages call it vhtoolkitUnity.

### Speech input
- **AcquireSpeech**: captures speech / typed text from the user.
- **PocketSphinx**: open-source speech recognition bundled with the toolkit.

## 4. Key markup languages to know

| Language | Role |
|----------|------|
| BML (Behavior Markup Language) | Describes *what* the body should do and when; NVBG → SmartBody |
| FML (Function Markup Language) | Communicative intent (used in the SAIBA model that BML belongs to) |
| PML (Perception Markup Language) | Sensed user state from MultiSense |

## 5. Getting started (per the official Getting Started page)

1. Go to the **Download** page and obtain the toolkit (Windows release).
2. Follow the **Tutorials** for installation.
3. Run the **example scenario** first before changing anything.
4. Then explore: Architecture → Components → FAQ → Support/Contact.

Site sections: Download, Architecture, Components, Tutorials, FAQ, Support,
Contact Us, License, Papers.

## 6. Licensing

- Free for **academic research** (and military use), provided all bundled
  component licenses are honoured (listed in Exhibit A of the agreement).
- Published research using the toolkit must **cite it** (license clause 3).
- **Commercial** use requires a separate license from ICT.
- It is research software: documentation and support are limited.
- A VHToolkit 2.0 (multi-platform, cloud services) has been announced as in
  development.

## 7. Questions to answer as I learn

- [ ] What exact VHMsg message types flow for one user turn?
- [ ] How is an NPCEditor classifier trained and evaluated?
- [ ] How does NVBG map text features to BML rules? Can rules be edited?
- [ ] Can SmartBody be driven from Python without the rest of the toolkit?
- [ ] What replaces PocketSphinx in VHToolkit 2.0?

## 8. Sources

- Project page: <https://ict.usc.edu/research/projects/vhtoolkit/>
- Overview PDF: <https://ict.usc.edu/wp-content/uploads/2021/09/Virtual-Human-Toolkit_Overview.pdf>
- Architecture doc: <https://vhtoolkit.ict.usc.edu/VHTK-doc/Architecture_14582040.html>
- Getting Started: <https://vhtoolkit.ict.usc.edu/VHTK-doc/Getting-Started_14582080.html>
- One-sheet (2023): <https://ict.usc.edu/wp-content/uploads/2023/09/VHTK-One-Sheet-Design-V1.pdf>
- DTIC paper on the toolkit: <https://apps.dtic.mil/sti/trecms/pdf/AD1157727.pdf>
