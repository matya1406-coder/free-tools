# Offline Obsidian link checker — first deliverable

This is the complete tool, not a specification. Copy the HTML code block below into a plain-text file named `vault-check.html`, save it as UTF-8, and open it in a current desktop browser. No installation, server or internet connection is needed. Select your Markdown files (or use the folder picker where supported), run the check, then optionally export CSV. Nothing is written to your notes.

## Assumptions and interpretation

- Select the complete vault for useful results. All conclusions are limited to readable selected `.md` files. Folder selection preserves paths for reporting; individual file selection may not.
- A note title is its filename without `.md`. Matching is Unicode NFC normalized and case-sensitive. Only simple `[[Title]]` links are resolved. Surrounding whitespace inside brackets is trimmed. Unique titles only: duplicates are ambiguous, never guessed.
- “No incoming links” means no supported link from another selected note; self-links do not count. It does not mean the note is unused or safe to delete.
- Mentions are exact title matches in remaining prose, bounded by characters other than Unicode letters, numbers, or underscore. They are suggestions; no fuzzy matching, stemming or frontmatter aliases. One finding per source line and target title; self-mentions are ignored. Existing linked text is excluded, but an unlinked occurrence elsewhere is still reported.
- YAML frontmatter at the start of a file, fenced code, inline code, HTML comments and Obsidian comments are ignored using lightweight masking. This is not a full Markdown parser. Indented code and unusual nested syntax may cause false positives. Heading text and ordinary prose are scanned.
- Path links, aliases, heading/block links, embeds and ordinary Markdown links are counted as excluded, not resolved. Attachments, external URLs and Obsidian’s internal index are not checked. Unsupported links do not contribute incoming edges, so those findings require manual review.
- Processing stays in browser memory; CSV contains private titles and excerpts. Review it before sharing. Large vaults may take time or exceed browser memory; try a representative subset first, recognizing its incomplete scope.

```html
<!doctype html>
<html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>Local vault checker</title>
<style>body{font:16px/1.5 system-ui,sans-serif;margin:auto;padding:24px;max-width:1100px;color:#172e27;background:#f7f8f4}button,input{font:inherit;margin:8px 8px 8px 0;max-width:100%}button{padding:10px 16px}label{display:block}table{border-collapse:collapse;width:100%;background:white}th,td{text-align:left;border:1px solid #ccd5cb;padding:8px;vertical-align:top;overflow-wrap:anywhere} .scroll{overflow:auto}th{background:#e6eee4}#status{font-weight:bold}summary{cursor:pointer}a{color:#245b43}</style></head><body>
<h1>Local vault checker</h1><p>Choose Markdown files, then run a read-only check. No files leave this browser. Select your whole vault for meaningful missing-target results.</p>
<label>Choose files <input id="files" type="file" accept=".md,text/markdown" multiple></label>
<label>Or choose a folder (browser support varies) <input id="folder" type="file" webkitdirectory multiple></label>
<button id="run" disabled>Run check</button><button id="export" disabled>Export CSV</button>
<p id="status" role="status" aria-live="polite">No files selected.</p>
<details open><summary>Scope and limitations</summary><p>Only simple [[Note title]] links, matched to unique filenames without .md, case-sensitively. Duplicate titles are ambiguous. Self-links do not count as incoming links. Mentions are exact title matches in prose, one per line and target, excluding existing link text.</p><p>Paths, aliases, heading/block links, embeds, ordinary Markdown links and attachments are unsupported. Excluded link syntax is counted below. Frontmatter aliases are not used. Frontmatter, fenced/inline code and comments are skipped using a lightweight parser; indented code and complex Markdown may produce false positives. A note with no incoming links may be intentional. This tool does not repair the Obsidian sidebar. Partial selections and unsupported links make results incomplete.</p><p>Files are never modified. CSV includes note names and excerpts; keep it private. Very large vaults can exhaust browser memory. Switching file pickers replaces the selection. CSV cells starting with spreadsheet formula characters are prefixed with an apostrophe for safer import.</p></details>
<div class="scroll"><table><thead><tr><th>Finding</th><th>Source</th><th>Line</th><th>Target</th><th>Detail</th></tr></thead><tbody id="rows"></tbody></table></div>
<script>
'use strict';
const norm = s => s.normalize('NFC');
const blank = s => s.replace(/[^\r\n]/g, ' ');
function mask(text) {
  let s = text.replace(/^\uFEFF/, '');
  s = s.replace(/^---[^\S\r\n]*\r?\n[\s\S]*?\r?\n(?:---|\.\.\.)[^\S\r\n]*(?=\r?\n|$)/, blank);
  s = s.replace(/<!--[\s\S]*?(?:-->|$)|%%[\s\S]*?(?:%%|$)/g, blank);
  let fence = null;
  s = s.split('\n').map(line => {
    const m = line.match(/^ {0,3}(`{3,}|~{3,})/);
    if (fence) {
      if (m && m[1][0] === fence[0] && m[1].length >= fence.length && line.slice(m[0].length).trim() === '') fence = null;
      return blank(line);
    }
    if (m) { fence = m[1]; return blank(line); }
    return line;
  }).join('\n');
  return s.replace(/(`+)[\s\S]*?\1/g, blank);
}
function audit(notes) {
  const rows = [], titles = new Map(), incoming = new Set();
  let excluded = 0;
  notes.forEach((n,i) => { n.title = norm(n.name.replace(/\.md$/i,'')); if (!titles.has(n.title)) titles.set(n.title,[]); titles.get(n.title).push(i); });
  const add = (kind, source, line, target, detail) => rows.push({kind,source,line,target,detail});
  for (const [title, ids] of titles) if(ids.length > 1) add('Duplicate title', ids.map(i=>notes[i].path).join(' ; '), '', title, 'Ambiguous: links and mentions to this title cannot be resolved.');
  notes.forEach((n,i) => {
    const clean = mask(norm(n.text));
    const prose = clean.split('\n').map((line,l) => {
      let s = line.replace(/!?\[\[([^\]\n]*)\]\]/g, (whole, inside) => {
        const target = inside.trim();
        if (whole.startsWith('!') || !target || /[|#^/\\]/.test(target) || /\.md$/i.test(target)) { excluded++; return blank(whole); }
        const ids = titles.get(target);
        if (!ids) add('Missing target',n.path,l+1,target,'No matching title in selected files.');
        else if (ids.length > 1) add('Ambiguous link',n.path,l+1,target,'Multiple notes have this title.');
        else if (ids[0] !== i) incoming.add(ids[0]);
        return blank(whole);
      });
      s = s.replace(/!?\[[^\]\n]*\](?:\([^\n]*?\)|\[[^\]\n]*\])/g, whole => { excluded++; return blank(whole); });
      // Reference definitions, shortcut references and autolinks are excluded too.
      if (/^ {0,3}\[[^\]]+\]:/.test(s)) { excluded++; return blank(s); }
      s = s.replace(/\[[^\]\n]+\]|<[^>\n]+>|https?:\/\/[^\s<>]+/g, whole => { excluded++; return blank(whole); });
      return s;
    });
    for (const [title,ids] of titles) {
      if (ids.length !== 1 || ids[0] === i || !title) continue;
      prose.forEach((line,l) => {
        let pos = line.indexOf(title), found = false;
        while (pos !== -1) {
          const before = Array.from(line.slice(0,pos)).pop() || '';
          const after = Array.from(line.slice(pos+title.length))[0] || '';
          if (!/[\p{L}\p{N}_]/u.test(before) && !/[\p{L}\p{N}_]/u.test(after)) { found = true; break; }
          pos = line.indexOf(title,pos+1);
        }
        if (found) add('Unlinked mention',n.path,l+1,title,line.trim().slice(0,220));
      });
    }
  });
  notes.forEach((n,i) => { if (!incoming.has(i)) add('No incoming links',n.path,'',n.title,'No supported link from another selected note; review before acting.'); });
  return {rows,excluded};
}
function csvCell(value) {
  let s = String(value ?? '');
  if (/^[\s]*[=+@-]/.test(s) || /^[\t\r\n]/.test(s)) s = "'" + s;
  return '"' + s.replace(/"/g,'""') + '"';
}
function csv(rows) {
  return '\uFEFF' + [['Finding','Source','Line','Target','Detail'],...rows.map(r=>[r.kind,r.source,r.line,r.target,r.detail])].map(r=>r.map(csvCell).join(',')).join('\r\n');
}
const byId = id => document.getElementById(id);
let selected = [], report = [];
function select(event) {
  selected = Array.from(event.target.files).filter(f=>/\.md$/i.test(f.name));
  byId(event.target.id === 'files' ? 'folder' : 'files').value = '';
  report = []; byId('rows').replaceChildren(); byId('export').disabled = true;
  byId('run').disabled = !selected.length;
  byId('status').textContent = selected.length + ' Markdown files selected. Previous results cleared.';
}
byId('files').onchange = select; byId('folder').onchange = select;
byId('run').onclick = async () => {
  for (const id of ['run','files','folder','export']) byId(id).disabled = true;
  byId('status').textContent = 'Reading selected files…';
  byId('rows').replaceChildren(); report = [];
  try {
    const notes = [], errors = [];
    for (const file of selected) {
      const path = file.webkitRelativePath || file.name;
      try { notes.push({name:file.name,path,text:await file.text()}); }
      catch { errors.push({kind:'Read error',source:path,line:'',target:'',detail:'Could not read file. Report scope is incomplete.'}); }
    }
    const result = audit(notes); report = [...errors,...result.rows];
    const frag = document.createDocumentFragment();
    for (const r of report.slice(0,2000)) {
      const tr = document.createElement('tr');
      for (const key of ['kind','source','line','target','detail']) { const td = document.createElement('td'); td.textContent = r[key]; tr.append(td); }
      frag.append(tr);
    }
    byId('rows').append(frag);
    byId('status').textContent = `${notes.length} files read; ${errors.length} read errors; ${result.excluded} excluded syntax occurrences; ${report.length} findings. Showing up to 2,000 rows; CSV includes all findings. Review scope limitations above.`;
    byId('export').disabled = false;
  } catch (e) { report = []; byId('status').textContent = 'Check failed. Try fewer files. ' + e.message; }
  finally { for (const id of ['run','files','folder']) byId(id).disabled = false; }
};
byId('export').onclick = () => {
  const url = URL.createObjectURL(new Blob([csv(report)],{type:'text/csv;charset=utf-8'}));
  const a = document.createElement('a'); a.href = url; a.download = 'vault-report.csv'; a.click();
  setTimeout(()=>URL.revokeObjectURL(url),1000);
};
</script></body></html>
```

## Small reproducible check

Select these three invented Markdown notes:

| Filename | Contents (each `↵` is a newline) |
|---|---|
| Home.md | `[[Topic]]↵[[Missing]]↵Topic appears again.↵[[Home]]` |
| Topic.md | `Home is mentioned here.` |
| Alone.md | `Just a standalone note.` |

Expected: one missing target (`Missing`, Home line 2); two unlinked mentions (`Topic`, Home line 3, and `Home`, Topic line 1); no incoming links for Home and Alone. Topic has an incoming link. Home’s self-link does not rescue it. CSV contains the same five findings.

To check ambiguity, add a second Topic.md in another folder: a duplicate-title finding and ambiguous Home link should appear, and mentions of Topic should no longer be suggested. For exclusions, place `[[Missing]]` inside fenced code or frontmatter: it should not be reported. A `[[Topic|label]]` link increments the excluded count, without resolving an incoming edge.


---

**Need it customised, or a full version for your work?** [Message us on Telegram](https://t.me/mostafa_ideas_2050_bot?start=r_page_2), no sign-up.

**محتاج نسخة مخصصة أو كاملة لشغلك؟** [ابعتلنا على تليجرام](https://t.me/mostafa_ideas_2050_bot?start=r_page_2) من غير تسجيل.
