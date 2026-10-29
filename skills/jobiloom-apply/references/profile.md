# Manage the applicant profile

## Choose the workspace

Use `.jobiloom/` inside the user's chosen Jobiloom workspace. Default to the current workspace if it is separate from the plugin's source repository and installed plugin directory. If running inside either plugin directory, ask the user to open or name a separate workspace before collecting personal information. Reuse that same workspace in future sessions and both clients. Do not silently search other folders for profiles.

Use normal filesystem tools to create and update the directory and files. A skill does not grant filesystem permissions. If access is denied, explain the blocked path and request an allowed workspace or the client's normal directory permission. Do not weaken sandbox settings, change global client configuration, or silently move the profile.

Before saving personal data, tell the user the directory and that the profile and documents persist locally for future applications. Offer session-only use if they do not want persistence. Session-only use must not create or update a profile or durable document copies. Use only files and facts supplied in that session. Place requested cover-letter drafts in a user-approved temporary location instead of `.jobiloom/`.

## Onboard once

Begin this workflow when Jobiloom is invoked without a ready profile and resume, even if no task follows the skill name. Guide the user through setup rather than offering a task menu. Preserve the original request and continue it only after readiness is established.

A profile is ready when the user has confirmed their name and email and a resume PDF exists and is readable. For saved profiles, verify the file named by `resume.file`. For session-only use, verify the supplied file and confirm the same facts in the conversation. Phone, location, work authorization, and other facts can remain unknown until required by a form. Optional demographic preferences and a cover-letter template never block readiness. Do not mark setup complete while a required fact is unconfirmed or the resume is missing or unreadable.

1. Read an existing `profile.json` before asking questions and check its saved resume. If the profile or resume is missing, request a resume PDF attachment or accessible file path. Reuse a readable saved resume when completing a partial profile. The browser must have access to the actual file for applications, not only extracted chat text. If an attachment is inaccessible, ask for an accessible file. Do not invent or recreate the resume from memory.
2. Read the resume and extract the available name, contact details, links, employment, and education. If the PDF cannot be read, request a readable copy or ask for missing facts.
3. Present the name, email, and resume path together for confirmation. Ask for a missing name or email in the same exchange. Include the storage location and session-only choice if not already settled. Wait for confirmed name and email before marking the profile ready. Leave other facts unknown until a form requires them or the user requests profile customization. Do not add questions about demographics, cover-letter templates, or remembering future answers to this setup exchange.
4. For persistent use, save confirmed facts to `profile.json` and copy the actual resume to `resume.pdf` inside `.jobiloom/` without removing the original. Leave `remember_new_answers` as `null` unless the user has chosen a preference. Save a supplied template as `cover-letter-template` with its original extension. Verify readable files and valid JSON, then report the saved location. For session-only use, retain confirmed facts in the conversation and use the supplied resume without durable copies. Once ready, continue the original request. If the skill was invoked alone, ask for a job title or link.

Create the directory with owner-only access and data files with owner-only read/write access where the operating system supports it. Save applicant facts and documents, not credentials, browser cookies, or session tokens. Do not copy the profile into the plugin package or submit it to a form.

## Store the profile

Use this versioned shape. Initialize unknown scalar values as `null` and collections as empty. This is a structural example, not applicant data:

```json
{
  "schema_version": 1,
  "resume": { "file": "resume.pdf" },
  "cover_letter_template": null,
  "remember_new_answers": null,
  "person": {
    "given_name": null,
    "family_name": null,
    "preferred_name": null,
    "email": null,
    "phone": null,
    "location": null,
    "links": []
  },
  "education": [],
  "employment": [],
  "answers": []
}
```

Store education entries with institution, degree, field of study, and dates when known. Store employment entries with employer, title, dates, and confirmed experience. Preserve compound names and international contact details as supplied instead of guessing a split or format.

When supplied, set `cover_letter_template` to an object with `file` pointing to its copied filename. Keep stored document paths relative to `.jobiloom/` and limited to files inside it. Resolve the absolute file path when a browser upload requires one.

For each reusable answer, store `question`, `answer`, `scope`, `source`, and `confirmed_at`. Store `scope` as an object with `country`, `company`, and `role`, using `null` for dimensions that do not restrict the answer. Authorization for one country must name that country. Demographic disclosure preferences belong here too. Use `source: "user"` and the actual confirmation date. Employer-specific answers must name the employer and must not be applied elsewhere.

Only persist confirmed applicant facts. Resume extraction is a draft until confirmed. A missing answer means unknown. An explicit non-disclosure choice is a saved preference. Generated cover-letter wording is job-specific prose, not a newly confirmed fact.

## Learn missing answers

Read the form question and choices, then compare their meaning and scope with saved facts. Reuse an answer only when compatible. For a missing required fact, ask a concise question with the available choices, wait, and continue the current application.

Ask about gender, race or ethnicity, disability, and veteran status only when a form requires an answer or the user requests disclosure preferences. Optional form questions can remain blank. Never infer these facts from a name, image, location, or resume. A skipped question remains unknown. Select a non-disclosure answer only when the user has chosen that preference.

Respect `remember_new_answers: true` by saving new reusable facts automatically. If false, use them for the current application only unless the user asks to save them. If the choice is missing, ask whether to remember a new answer before persisting it. Respect instructions to use an answer only once. Save education and employment corrections in their corresponding collections rather than duplicating them as unrelated answers.

Before updating, reread the latest file and preserve unrelated fields. Write valid JSON to a temporary file in the same directory, then replace `profile.json` atomically. If another session changed the same fact, resolve the conflict with the user instead of overwriting it. If the JSON is invalid or the schema version is unsupported, preserve the file and report the issue. Never silently reset an unreadable profile or report a failed save as successful.

## Update or remove data

On request, show relevant saved fields, correct facts, replace the resume or template, or remove the profile. Review changed facts from a new resume before replacing confirmed data. An application-specific document override does not replace the default unless requested.

Delete only requested Jobiloom-owned files. Preserve original supplied documents and unrelated workspace files. For a request to delete the entire profile, remove its JSON, copied resume, copied template, and generated cover-letter files, then verify removal. Do not recreate deleted data until the user requests setup or another application.
