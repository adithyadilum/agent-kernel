# Guardrail configuration

Two deliberate omissions here, both load-bearing.

## No PII detection

`Contains PII` is not enabled on either path, and `PHONE_NUMBER` in particular is not
listed anywhere. A mother types her assigned PHM's phone number during registration, and
the escalation path needs that number to deliver. A PII guardrail that blocked or redacted
it would break the system in the name of protecting it.

PII redaction in this project applies to **log output only**, implemented as a
`logging.Filter` in `redaction.py`. Agent Kernel has no built-in log redaction.

## Narrow moderation categories

The input moderation categories are restricted to the four that cannot plausibly fire on a
maternal symptom report:

- `sexual/minors`, `hate/threatening`, `harassment/threatening`, `illicit/violent`

`self-harm` and `violence/graphic` are deliberately **excluded**. A mother describing heavy
bleeding, severe pain, or a baby that has stopped moving is the single most important
message this service can receive. If an input guardrail blocked it, she would get a generic
rejection, `screen_danger_signs` would never run, and no escalation would happen. The whole
fail-toward-escalation design would be bypassed by the safety layer.

That is a real trade-off, not an oversight: this configuration accepts a narrower moderation
net in exchange for never silencing a symptom report. See Known Limitations in the project
README.

## Model configuration

`MATHRU_MODEL` only controls the agents. Guardrail wrapper models come from `config.yaml`
(or `AK_GUARDRAIL__INPUT__MODEL` / `AK_GUARDRAIL__OUTPUT__MODEL`). The Jailbreak and NSFW
Text checks each have a separate literal `config.model` in `input.json` and `output.json`.
Change all four guardrail settings explicitly when switching the chat model; JSON does
not interpolate environment variables. Moderation uses its own moderation service.
