---
title: "FormFill privacy policy"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 15
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# FormFill privacy policy

**English** | [简体中文](16-FormFill_隐私政策.md)

> **Archive status:** FormFill stopped productization and was frozen on 2026-11-18. The project was never published in a browser store or deployed to a production website. This policy describes the archived code snapshot whose `package.json` version is still `0.1.0`, but which contains the unreleased `0.2.0` behavior; version `0.2.0` was never published.

FormFill fills web forms using a model service selected and configured by you. This policy describes the archived code, including the pause of automatic learning. See the [product wrap-up and handoff report](22-FormFill_产品收尾与交接报告.md) (Chinese).

## Data processed and sent

When you request a fill, the extension reads the active page's standard form controls, including controls outside a `<form>`. It sends your description and a structured form summary directly to your configured model endpoint. The summary includes:

- Control tag and type, name, ID, label, placeholder, CSS class, title, and accessibility label (`aria-label`), when present.
- Select option text and values, and the value attributes of radio buttons and checkboxes.
- Visible descriptive text from up to three parent elements, excluding form-control contents, editable regions, executable markup, and hidden content; whitespace is normalized. `content.ts` keeps the first 120 characters and may append `...`; `llm-retry.ts` limits this field to **120 characters before sending**.
- A form signature derived from tag, type, name, ID, and label. At most 200 controls are included in a model request.

The summary does not contain full page HTML or ordinary input/textarea values as separate value fields. This is **not a blanket exclusion of sensitive data**: labels, options, other visible page text, your description, and files may contain personal or sensitive information. Password, hidden, disabled (including disabled fieldsets), read-only, and certain non-entry controls are excluded; ancestor visibility is checked. The same protection applies before writing. These controls are excluded by the scanner's checks; those checks do not guarantee that all sensitive page text is removed.

Model suggestions are shown as a preview in the popup first, and nothing is written until you apply. In the preview, each suggested value is editable and shows the control's current value next to it; fields with existing values are unchecked by default. Existing values are used only for this local preview comparison and are not sent to the model. If the page structure changes between preview and apply, nothing is written. After applying, undo restores only fields written by that fill that you have not edited since; the undo snapshot lives in the page's content script memory and is lost when the page reloads.

If you import a file, it is parsed locally. A fill request sends the filename and up to 30,000 characters of parsed content, plus a truncation note if needed. Files are limited to 10 MB. For Excel, you pick the worksheet, header row, and one record; only the selected record's values are included in the request, and other rows are never sent. Input and parsed file content are held in popup memory rather than saved as an input/file archive by FormFill. Closing the popup discards that popup state.

Saved historical values are never added to requests. The prompt tells the model to ignore instructions inside form summaries and files; this instruction does not guarantee that a model will resist every injection.

Requests go to the **provider/base URL you select**, including any custom endpoint or local Ollama service. They do not pass through developer-operated servers. Provider handling and retention are governed by that provider's terms. The provider sees network request metadata such as your IP address; a local service may in turn use remote infrastructure according to its own configuration.

**Test connection** sends a small fixed prompt using your configured model and authentication, without a form summary, file, or learning sample.

## Local storage and learning

Settings, provider profiles, API keys, and privacy-notice acceptance are stored in the browser extension's local storage, not browser sync storage. API keys are stored locally and sent to the configured endpoint for authentication; FormFill does not upload them to a developer service or encrypt them separately. Switching providers retains their individual profiles.

If you confirm saving a field mapping after an Excel fill, the extension stores the site origin, a form structure signature, and "column name → field location" pairs in local storage (at most 20 mappings). Mappings never contain record values, page text, or API keys, and are never sent to a model or any server. Reusing a mapping reads only the currently selected record and still requires preview and apply. You can delete mappings from the popup.

Automatic learning is **paused in this version**. The extension does not attach submission listeners, collect new samples, or include saved history in model requests. Storage schema is v6: the v5 migration forcibly disabled the old learning flag without changing saved provider configuration, and v6 added the optional reasoning-effort setting. There is no switch to turn learning back on. After updating, **reload existing web pages** so their older content scripts are replaced.

Old samples remain in extension local storage until you clear them, inspect them after expiry, or uninstall the extension. They contain the origin, form signature, time, and field keys/values (some older records may also contain a path). **View local records** shows their actual contents only inside the popup. Viewing removes malformed records and records older than **90 days**, and keeps at most the **50** most recent records; this is not scheduled cleanup. **Clear learning records** deletes all samples. Viewing does not contact a model endpoint. Legacy filtering was heuristic; saved records may contain personal information. Local deletion cannot delete copies a provider may already hold.

## What FormFill does not do

FormFill has no analytics or advertising, does not sell data, and has no developer-operated collection server. It does not track general browsing history, keystrokes, or clicks for analytics, and does not automatically submit forms. It processes the active form when you request filling; it does not observe form submissions. Information you supply or that appears in form context may still be sent to your selected provider as described above.

## Permissions

- `storage`: local settings, API keys, provider profiles, and remaining legacy samples.
- `activeTab`: access to the active tab after user interaction, so the popup can target the page to fill.
- Content script matching `<all_urls>`: make form scanning, filling, and highlighting available across supported sites. The script loads on matching pages; a form summary is requested for model use when you invoke filling. This is broad page access, not on-demand script injection. Browser-restricted pages remain unavailable.

## Contact

Report issues and privacy requests through the project's issue tracker at <项目归档仓库>. The in-extension **Feedback** button opens that page; FormFill does not send email or submit anything on your behalf. Issues are public: do not paste API keys, personal documents, or real form values.
