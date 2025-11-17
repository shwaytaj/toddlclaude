---
name: blog-writer
description: Write AEO-optimized blog articles for toddl.health following 2025 best practices. Use when creating health/parenting blog content that needs Answer Engine Optimization (AEO), featured snippet formatting, E-E-A-T signals, and parent-friendly tone. Handles complete blog structure from title to medical disclaimer.
---

# Blog Writer for toddl.health

Create complete, publication-ready blog articles optimized for AI answer engines, search engines, and parent readers.

## Process

### 1. Understand the Topic

Ask the user for:
- **Blog title or topic** (required)
- **Target keyword** (optional - extract from title if not provided)
- **Target audience specifics** (optional - default: first-time mothers, anxious parents)
- **Word count target** (optional - default: 1,500-2,000 words)

### 2. Load Guidelines

Always read the complete guidelines first:

```bash
view /home/claude/blog-writer/references/Blog_Structure.md
```

This contains all formatting rules, AEO requirements, and examples.

### 3. Research (if needed)

For current medical guidelines, statistics, or recent developments:
- Use web_search for latest CDC, WHO, AAP, NHS recommendations
- Verify statistics from authoritative sources
- Note publication dates for citations

### 4. Create Complete Article

Generate the full blog article following the structure from Blog_Structure.md:

**Required Elements:**
1. YAML frontmatter (title, description, dates, author, tags, etc.)
2. Structured data (Article and FAQPage schemas)
3. H1 title (clear, question-based or benefit-driven, <60 chars)
4. E-E-A-T signals (credentials, dates, medical reviewer)
5. Warm introduction (2-4 sentences, <100 words)
6. Key Takeaways box (3-6 bullets with 1-2 citations)
7. Question-style H2 headings (from "People Also Ask" patterns)
8. Quick Answers (40-60 words after each H2)
9. Supporting detail (2-4 sentence paragraphs, tables, lists)
10. FAQ section (5-8 Q&As)
11. Related Reading section
12. Medical Disclaimer
13. Medical Review & Sources section
14. Author bio
15. Newsletter CTA

**Critical Formatting:**
- Paragraphs: 2-4 sentences max (40-60 words)
- Quick Answers: 40-60 words, standalone, cited
- Tables: Use for comparisons, timelines, data
- Lists: Bullets for 3+ items, numbered for sequences
- Callout boxes: For critical safety info
- All statistics must include source citations

### 5. Deliver as Markdown File

Create the article as a .md file in /mnt/user-data/outputs/ with:
- Filename: `{kebab-case-title}.md`
- Complete structure from frontmatter to author bio
- All required sections properly formatted

## Key Principles

**Tone:** Warm, empathetic, evidence-based, reassuring
**Audience:** Primarily first-time mothers experiencing parenting anxiety
**Reading Level:** Grade 8 or below (use simple language)
**Authority:** Medical credentials, current citations, expert review
**Optimization:** AEO for AI answer engines, featured snippets, voice search

## Common Pitfalls to Avoid

- Don't skip the Key Takeaways box (critical for AEO)
- Don't write paragraphs longer than 4 sentences
- Don't forget Quick Answers after H2 headings
- Don't use generic H2s (must be natural questions)
- Don't cite sources without dates
- Don't skip medical disclaimer
- Don't forget structured data schemas
- Remove any m dashes.
- Do not add any information about any specific author or any medical reviewer. 
- Dont mention that the blog was reviewed by any medical reviewer.
- Do not have any marketing call to action that asks the user to subscribe
- Dont have any mention of toddl.health
- Don't render any actual json schemas or any scripts in the blog content
- Don't include any lines about any conflict of interest
- Remove any mentions of "Last Updated"

## Content Safety

For medical content:
- Defer to professional medical judgment for all diagnoses
- Include "when to seek immediate medical attention" warnings
- Never provide specific medication dosing (refer to pediatrician)
- Emphasize consulting healthcare providers
- Include comprehensive medical disclaimer

## Quality Checklist

Before delivery, verify:
- [ ] All H2s are natural language questions
- [ ] Each H2 has 40-60 word Quick Answer
- [ ] Key Takeaways box present after introduction
- [ ] FAQ section has 5-8 questions
- [ ] All stats have source citations with dates
- [ ] Medical disclaimer included
- [ ] Author credentials displayed
- [ ] Paragraphs are 2-4 sentences
- [ ] Structured data schemas included
