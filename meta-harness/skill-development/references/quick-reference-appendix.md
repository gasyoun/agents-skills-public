## Quick Reference

### Minimal Skill

```
skill-name/
└── SKILL.md
```

Good for: Simple knowledge, no complex resources needed

### Standard Skill (Recommended)

```
skill-name/
├── SKILL.md
├── references/
│   └── detailed-guide.md
└── examples/
    └── working-example.sh
```

Good for: Most plugin skills with detailed documentation

### Complete Skill

```
skill-name/
├── SKILL.md
├── references/
│   ├── patterns.md
│   └── advanced.md
├── examples/
│   ├── example1.sh
│   └── example2.json
└── scripts/
    └── validate.sh
```

Good for: Complex domains with validation utilities

## Best Practices Summary

✅ **DO:**
- Use third-person in description ("This skill should be used when...")
- Include specific trigger phrases ("create X", "configure Y")
- Keep SKILL.md lean (1,500-2,000 words)
- Use progressive disclosure (move details to references/)
- Write in imperative/infinitive form
- Reference supporting files clearly
- Provide working examples
- Create utility scripts for common operations
- Study plugin-dev's skills as templates

❌ **DON'T:**
- Use second person anywhere
- Have vague trigger conditions
- Put everything in SKILL.md (>3,000 words without references/)
- Write in second person ("You should...")
- Leave resources unreferenced
- Include broken or incomplete examples
- Skip validation

## Additional Resources

### Study These Skills

Plugin-dev's skills demonstrate best practices:
- `../hook-development/` - Progressive disclosure, utilities
- `../agent-development/` - AI-assisted creation, references
- `../mcp-integration/` - Comprehensive references
- `../plugin-settings/` - Real-world examples
- `../command-development/` - Clear critical concepts
- `../plugin-structure/` - Good organization

### Reference Files

For complete skill-creator methodology:
- **`references/skill-creator-original.md`** - Full original skill-creator content

## Implementation Workflow

To create a skill for your plugin:

1. **Understand use cases**: Identify concrete examples of skill usage
2. **Plan resources**: Determine what scripts/references/examples needed
3. **Create structure**: `mkdir -p skills/skill-name/{references,examples,scripts}`
4. **Write SKILL.md**:
   - Frontmatter with third-person description and trigger phrases
   - Lean body (1,500-2,000 words) in imperative form
   - Reference supporting files
5. **Add resources**: Create references/, examples/, scripts/ as needed
6. **Validate**: Check description, writing style, organization
7. **Test**: Verify skill loads on expected triggers
8. **Iterate**: Improve based on usage

Focus on strong trigger descriptions, progressive disclosure, and imperative writing style for effective skills that load when needed and provide targeted guidance.
