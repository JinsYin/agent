---
name: x-zh-to-en
description: Convert Chinese text into natural English with light interpretation and improvement. Use when the user writes in Chinese and wants an English version that preserves intent but can add clarifications, suggestions, or slight enhancements rather than a literal translation.
---

# Chinese to English (x-zh-to-en)

## Instructions

When this skill is active, follow these guidelines for every request:

1. **Determine intent**
   - Read the Chinese input and identify the user's core intent, tone, and audience.
   - Notice whether it is casual chat, documentation, email, marketing copy, technical writing, or something else.

2. **Translate with understanding, not literally**
   - Convert the Chinese into **fluent, natural English** rather than word-for-word translation.
   - Preserve the original meaning and tone while making phrasing sound native.
   - You **may slightly rephrase, compress, or expand** if it helps clarity or flow, as long as intent stays the same.

3. **Add helpful interpretation**
   - Where the original text is implicit or context-dependent, you may:
     - Make implicit assumptions explicit.
     - Clarify vague references (e.g. "this thing" → briefly specify what it likely refers to).
     - Smooth over culture-specific idioms with equivalent English expressions.
   - If there are multiple plausible interpretations, pick the one that fits best and keep the English concise.

4. **Improve quality**
   - Fix grammar, structure, and wording to be clear and professional when appropriate.
   - Adjust formality level to match the original text (informal, neutral, formal).
   - Avoid exaggerated or flowery language unless clearly requested by the tone of the Chinese.

5. **Add suggestions when useful**
   - When the Chinese text is clearly asking for "better wording" or "help me phrase this", you may:
     - Provide **one main recommended English version**, and
     - Optionally offer **1–2 alternative phrasings** if they add real value (e.g. more formal vs more friendly).
   - Keep suggestions focused; do not overwhelm with too many variants.

6. **Stay in English**
   - The output should be entirely in English unless the user explicitly asks for bilingual output.
   - Do not repeat the original Chinese unless the user asks for side-by-side comparison.

7. **No extra explanation by default**
   - By default, just output the improved English (plus any brief alternatives if appropriate).
   - Only explain translation choices when the user explicitly asks for an explanation or learning.

## Examples

### Example 1: Simple sentence

Input (Chinese):
- "帮我把下面这段话翻译成英文，可以稍微优化一下表达：我们这次主要目标是提升用户在移动端的体验。"

Output (English):
- "Our main goal this time is to improve the user experience on mobile devices."

### Example 2: With interpretation and refinement

Input (Chinese):
- "请帮我写一段英文介绍，说明我们新产品的定位：主要面向中小企业，帮助他们更简单地管理合规风险。语气专业但不要太生硬。"

Possible output (English):
- "Our new product is designed for small and medium-sized businesses, helping them manage compliance risks in a simpler and more efficient way. The tone is professional yet approachable, making it easier for teams to adopt in their day-to-day work."

### Example 3: Multiple suggestions

Input (Chinese):
- "这句话帮我润色成英文，用在对外的项目介绍里：这个功能可以让开发者更快地集成我们的 SDK。"

Output (English):
- "This feature enables developers to integrate our SDK much faster."
  
Alternative:
- "This feature helps developers integrate our SDK more quickly and with less effort."


