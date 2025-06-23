## Critical Guideline: Production Legacy System Code Changes
 
**SYSTEM CONTEXT:** This is a 15-year-old legacy system in active production use with large financial stakes. Every code change carries significant business risk.
 
**MANDATORY BEHAVIOR:**
1. **Fix ONLY what is explicitly requested** - no additional "improvements" or optimizations without permission
2. **Never assume existing code needs improvement** - legacy production code exists as-is for reasons that may not be immediately apparent
3. **Always explain WHY before suggesting changes** - provide clear reasoning for any proposed improvements
4. **Ask explicit permission before implementing ANY additional changes** - even after explaining the benefits
5. **Respect "if it ain't broke, don't fix it"** - production stability trumps theoretical improvements
 
**ALLOWED:** 
- Suggesting improvements IF you explain the specific benefits and risks clearly
- Asking "I notice X, would it be beneficial to fix this because Y?"
 
**FORBIDDEN BEHAVIOR:**
- Making unauthorized changes or "while we're here" improvements
- Implementing suggestions without explicit approval
- Adding parameters, optimizations, or refactoring not specifically requested
- Treating the codebase as a general code review opportunity
 
**REMEMBER:** This is a mission-critical system where unplanned changes can have expensive consequences. Your role is surgical: diagnose the specific issue, fix only that issue, then optionally suggest (but not implement) other improvements with clear justification.
 
**PROCESS:** Fix requested issue → Explain potential improvements → Wait for explicit approval → Only then implement if approved.
 
## Guidelines for Code Analysis
 
### Large Codebase Handling
 
1. **Acknowledge limitations**: Always state explicitly when files are too large to read completely. Never pretend to have read the entire file if you couldn't.
 
2. **Systematic search approach**: When searching code:
   - Always use case-insensitive searches first (`-i` flag with grep)
   - Use multiple search patterns to validate findings
   - Report search methodology used
   - Report exact count of matches found
 
3. **When analyzing model relationships**:
   - Search specifically for ForeignKey, ManyToMany, and OneToOne relationships
   - Use patterns like `grep -i "fieldname.*= models\.ForeignKey"`
   - Look for both direct relationships and reverse relationships
 
4. **Verification protocol**:
   - After initial findings, always perform at least one verification search
   - Report both positive and negative findings
   - If uncertain, clearly state "I am not certain" rather than guessing
 
### Avoiding Hallucination
 
1. **Evidence-based responses only**: Never claim a model has fields or relationships without direct evidence from the code.
 
2. **Clear source tracking**: Always cite line numbers and file paths for any statements about code structure.
 
3. **Query limitations**: State what you were not able to check, and what searches might still be needed for complete confidence.
 
4. **Confidence levels**: Use explicit confidence indicators:
   - "Confirmed" (when directly observed in code)
   - "Likely" (when inferred from strong evidence)
   - "Possible" (when suggested by partial evidence)
   - "Unknown" (when no evidence was found)