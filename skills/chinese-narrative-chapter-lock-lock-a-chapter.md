---
name: Lock a chapter's terminology for CN→EN webnovel translation
description: >-
  Paste a Chinese webnovel chapter and get back locked genre terms, title-specific
  entities, a character bible, and a source,target glossary CSV ready for DeepL/Crowdin
  or any CAT tool — so names, honorifics, ranks and cultivation stages stay consistent
  across chapters.
api: openapi/chinese-narrative-chapter-lock-openapi.json
operations: [lock_endpoint_v1_lock_post, health_health_get]
generated: '2026-09-05'
method: generated
source: >-
  Generated from openapi/chinese-narrative-chapter-lock-openapi.json and the provider's
  worked example at https://culturebiz-xianxia-lock.onrender.com/docs. Every operationId
  is verified verbatim in the spec.
---

# Lock a chapter's terminology

## When to use

You are translating (or AI-translating) a Chinese webnovel — xianxia/cultivation genre —
into English, and terminology drifts across a long work: a sect name, a character name, or
a cultivation rank gets rendered differently in chapter 40 than in chapter 4. This API locks
those terms per pasted chapter. It is NOT a machine-translation engine and does not host
novels; send only text the buyer has the right to paste.

## Steps

1. (Optional) Verify the service is up with `health_health_get` — `GET /health` returns
   `{"status":"ok"}`.
2. Call `lock_endpoint_v1_lock_post` — `POST /v1/lock` with body
   `{"text": "<pasted Chinese chapter>"}` and `Content-Type: application/json`.
   The `text` field is required (see `#/components/schemas/LockRequest`); a missing or
   non-string value returns a 422 FastAPI validation envelope
   (see ../errors/chinese-narrative-chapter-lock-problem-types.yml).
3. Read the response:
   - `locked_terms[]` `{zh, en, pos}` — genre-convention terms (honorifics, ranks, energy
     vocabulary) with their locked English renderings. Use these as hard constraints in
     your MT prompt or CAT termbase.
   - `title_entities[]` `{zh, kind}` — title-specific names (person/sect/skill) extracted
     from THIS paste. Decide their English renderings once, then keep them.
   - `character_bible[]` `{zh, kind, count}` — recurring entities with occurrence counts;
     high-count entries are the ones worth naming carefully.
   - `csv` — a two-column `source,target` glossary string. Title entities pass through
     with `target = source` (untranslated) — fill in your chosen renderings before import.
4. Import the CSV into your CAT/MT tool (DeepL glossary, Crowdin glossary) and translate
   the chapter with the glossary applied.
5. Repeat per chapter. Consistency is per-paste: the service keeps no state between calls,
   so carry your growing glossary on your side and merge each chapter's new rows into it.

## Rules

- Direct calls to https://culturebiz-xianxia-lock.onrender.com need no authentication;
  metered access via RapidAPI requires the platform's `X-RapidAPI-Key`
  (see ../authentication/chinese-narrative-chapter-lock-authentication.yml).
- The call is a stateless transformation — safe to retry on timeout when calling the
  origin directly; a retry through RapidAPI consumes a plan request
  (see ../conventions/chinese-narrative-chapter-lock-conventions.yml).
- No rate-limit headers are returned; pace batch runs yourself
  (see ../rate-limits/chinese-narrative-chapter-lock-rate-limits.yml).
