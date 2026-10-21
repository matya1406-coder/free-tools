# A usable workflow for saving AI conversations in Obsidian

This is a first deliverable: a local formatter, a short operating guide, and a ready-to-copy note structure. The working formatter is embedded in `offer.html`; save that file and open it in a browser without an internet connection.

## Assumptions and limits

- You can copy the conversation as text. No actual conversation was supplied, so the example below is invented, not a customer transcript or a claimed result.
- For Voice Mode, first check whether your app exposes a text transcript for that conversation. If it does, copy the passages you want to keep. If it does not, this tool cannot recover or transcribe the audio; you can write your own recollection and label it as such.
- Input uses `Assistant:` on its own line before each answer, or another exact whole-line marker you choose. Speaker labels are not inferred. Inline `Assistant: answer` must be split into a marker line and an answer line.
- Supported headings start at the left margin with one to six `#` characters followed by whitespace or the end of the line. Top-level backtick and tilde code fences are protected. Underlined headings and nested Markdown are outside the normalization scope.
- This formats text; it does not verify the AI's claims, summarize, import chats, or connect ideas automatically. Keep the original if exact source formatting matters.

## One-page operating guide

1. Open `offer.html` locally in a modern browser. No installation or Obsidian plugin is needed.
2. Copy the available transcript or conversation text. Remove anything you do not want saved. Give the note a descriptive title, such as “Choosing headings for saved conversations”.
3. Put `Assistant:` on its own line immediately before each answer. Choose a different marker if that whole line already appears as ordinary conversation text. The separator is ignored inside supported code fences.
4. Paste the text, enter the same separator, and select **Build Markdown**. Text before the first marker becomes “Context / first question”; each marked answer starts a “Response N” section. Later questions remain at the end of the preceding response section. For question-and-answer sections instead, place a custom marker such as `=== TURN ===` before each question.
5. Read the Markdown preview. There is one H1 title, H2 sections and an Obsidian contents list. Answer headings move to H3–H6; the smallest heading level in each section becomes H3. Very deep levels may collapse at H6. Code-fence contents are unchanged, apart from line-ending normalization.
6. If you see a single “Conversation” section, no separator matched. Fix the marker if you wanted multiple sections. An unfinished code fence blocks the conversion so you can repair it.
7. Select **Download .md**, move the downloaded note into your vault, and open it in Obsidian. Use Outline to browse the responses. The contents links use Obsidian's internal heading-link syntax.
8. Add a short personal takeaway and links to existing notes only after reviewing the content. Searchable titles and meaningful links help retrieval; retaining a chat by itself does not establish useful connections.

Input changes clear the previous preview and disable download until you build again. The example button asks before replacing non-empty input. Nothing is saved automatically: download before closing or reloading the page. The tool makes no network requests and does not store input in browser storage; the downloaded file and any browser or vault backups are under your control.

## Worked input

Title: `Choosing headings for saved conversations`  
Separator: `Assistant:`

````text
User: How should I organize a saved conversation?
Assistant:
# Use a predictable outline
Give the note one title and each response its own section.
## Keep supporting detail beneath it
Use deeper headings for the details of that response.

User: Can a Markdown code example keep its original headings?
Assistant:
# Preserve the example
Yes: keep the example inside a fenced code block.
```markdown
# Original example title
## Original example section
Assistant:
```
## Review before saving
Check that the response boundaries match what you intended.
````

## Resulting note

The contents between the fences below are ready to paste into a new Obsidian note. The first answer contains the following question because the separator marks answers, not complete turns.

````markdown
# Choosing headings for saved conversations

**Contents**

- [[#Context / first question|Context / first question]]
- [[#Response 1|Response 1]]
- [[#Response 2|Response 2]]

## Context / first question

User: How should I organize a saved conversation?

## Response 1

### Use a predictable outline
Give the note one title and each response its own section.
#### Keep supporting detail beneath it
Use deeper headings for the details of that response.

User: Can a Markdown code example keep its original headings?

## Response 2

### Preserve the example
Yes: keep the example inside a fenced code block.
```markdown
# Original example title
## Original example section
Assistant:
```
#### Review before saving
Check that the response boundaries match what you intended.
````

## Reusable retrieval section

After checking a saved conversation, append this section and fill in the brackets. These placeholders are instructions for your own note, not extracted insights.

```markdown
## My retrieval notes

Captured on: [date]
Source: [typed chat / copied voice transcript / my recollection]
Search terms: [two or three phrases I would actually search for]

### My takeaway
[One useful point in my own words, after checking it.]

### Related notes
- [[Existing note title]] — [why this connection is useful]

### Still to verify
- [A claim or action that needs an independent check]
```

## Quick acceptance checks

- Load the built-in example: its `Assistant:` inside the code fence must not create a response.
- Both code headings must keep their original `#` levels; normal answer headings must move below H2.
- Change the separator to a marker absent from the input: the tool must report no match and produce one section.
- Remove a closing code fence: the tool must refuse conversion and explain why.
- Change input after conversion: the previous download must be disabled.
- Save the page locally, disconnect, open it and convert again. Download the note and check its outline and contents links in Obsidian.

---

**Need it customised, or a full version for your work?** [Message us on Telegram](https://t.me/mostafa_ideas_2050_bot?start=r_page_9), no sign-up.

**محتاج نسخة مخصصة أو كاملة لشغلك؟** [ابعتلنا على تليجرام](https://t.me/mostafa_ideas_2050_bot?start=r_page_9) من غير تسجيل.
