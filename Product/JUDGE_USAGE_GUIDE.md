# AI Recommendation Judge - Usage Guide

## Quick Start

### Prerequisites
1. Python 3.8 or higher
2. Anthropic API key
3. Required Python package: `anthropic`

### Installation

```bash
# Install the Anthropic SDK
pip install anthropic

# Set your API key as an environment variable
export ANTHROPIC_API_KEY='your-api-key-here'
```

### Basic Usage

#### 1. Running the Example Script

```bash
cd Product
python judge_evaluator.py
```

This will run an example evaluation and display results in the terminal.

#### 2. Evaluating Your Own Recommendations

```python
from judge_evaluator import RecommendationJudge

# Initialize the judge
judge = RecommendationJudge()

# Prepare your data
recommendations = [
    {
        "title": "Your recommendation title",
        "description": "Your recommendation description",
        "citations": [
            {
                "source": "CDC Developmental Milestones",
                "url": "https://www.cdc.gov/..."
            }
        ]
    }
    # ... more recommendations
]

milestone_context = {
    "name": "Milestone name",
    "description": "Milestone description",
    "ageRange": "4-6",
    "category": "Physical Development - Gross Motor"
}

child_info = {
    "correctedAge": 5,
    "chronologicalAge": 6,
    "birthDetails": "Born 4 weeks early",
    "medicalHistory": "None provided",
    "parentMedicalHistory": "None provided"
}

# Run evaluation
results = judge.evaluate_recommendations(
    recommendations=recommendations,
    milestone_context=milestone_context,
    child_info=child_info,
    recommendation_type="guide"  # or "toy"
)

# Access results
print(f"Pass rate: {results['summary']['passRate']}%")
for evaluation in results['evaluations']:
    print(f"{evaluation['recommendationTitle']}: {evaluation['overallAssessment']}")

# Save to file
judge.save_results(results, "my_evaluation_results.json")
```

---

## Use Cases

### Use Case 1: Pre-Launch Quality Assurance

**Goal:** Validate recommendations before deploying to production

```python
from judge_evaluator import RecommendationJudge
import json

# Load recommendations from your API response
with open('api_recommendations.json', 'r') as f:
    data = json.load(f)

judge = RecommendationJudge()
results = judge.evaluate_recommendations(
    recommendations=data['recommendations'],
    milestone_context=data['milestone'],
    child_info=data['childInfo'],
    recommendation_type="guide"
)

# Check if any failed
if results['summary']['failed'] > 0:
    print("WARNING: Some recommendations failed evaluation!")
    for eval in results['evaluations']:
        if eval['overallAssessment'] == 'FAIL':
            print(f"FAILED: {eval['recommendationTitle']}")
            print(f"Reason: {eval['overallReasoning']}")
            print(f"Actions: {eval['recommendedActions']}")
```

### Use Case 2: Batch Testing Across Age Ranges

**Goal:** Test recommendation quality across different developmental stages

```python
from judge_evaluator import RecommendationJudge
import json

# Define test cases for different age ranges
test_cases = [
    {
        "name": "Newborn (0-2 months)",
        "milestone": {...},
        "childInfo": {"correctedAge": 1, ...}
    },
    {
        "name": "Infant (4-6 months)",
        "milestone": {...},
        "childInfo": {"correctedAge": 5, ...}
    },
    # ... more test cases
]

judge = RecommendationJudge()
batch_results = []

for test_case in test_cases:
    print(f"Testing: {test_case['name']}")

    # Generate recommendations (your API call here)
    recommendations = generate_recommendations(test_case)

    # Evaluate
    results = judge.evaluate_recommendations(
        recommendations=recommendations,
        milestone_context=test_case['milestone'],
        child_info=test_case['childInfo'],
        recommendation_type="guide"
    )

    batch_results.append({
        "testCase": test_case['name'],
        "results": results
    })

    print(f"  Pass rate: {results['summary']['passRate']}%")

# Save comprehensive report
with open('batch_evaluation_report.json', 'w') as f:
    json.dump(batch_results, f, indent=2)
```

### Use Case 3: Monitoring Production Recommendations

**Goal:** Randomly sample and evaluate production recommendations

```python
from judge_evaluator import RecommendationJudge
import random

# Sample from production database
sampled_recommendations = sample_from_database(limit=50)

judge = RecommendationJudge()
failed_count = 0

for sample in sampled_recommendations:
    results = judge.evaluate_recommendations(
        recommendations=sample['recommendations'],
        milestone_context=sample['milestone'],
        child_info=sample['childInfo'],
        recommendation_type=sample['type']
    )

    if results['summary']['failed'] > 0:
        failed_count += 1
        # Log for review
        log_failed_recommendation(sample['id'], results)

        # Alert if critical safety issue
        for eval in results['evaluations']:
            if eval['evaluation']['childSafety']['score'] < 4:
                send_alert(f"Critical safety issue in recommendation {sample['id']}")

print(f"Production quality check: {failed_count}/{len(sampled_recommendations)} batches had failures")
```

### Use Case 4: Citation Verification

**Goal:** Verify that citations are accurate and URLs work

```python
from judge_evaluator import RecommendationJudge
import requests

def verify_urls(recommendations):
    """Check if citation URLs are accessible."""
    url_results = []

    for rec in recommendations:
        for citation in rec.get('citations', []):
            if 'url' in citation:
                try:
                    response = requests.head(citation['url'], timeout=5)
                    url_results.append({
                        'url': citation['url'],
                        'source': citation['source'],
                        'status': response.status_code,
                        'valid': response.status_code == 200
                    })
                except Exception as e:
                    url_results.append({
                        'url': citation['url'],
                        'source': citation['source'],
                        'error': str(e),
                        'valid': False
                    })

    return url_results

# First, verify URLs are accessible
url_check = verify_urls(recommendations)
broken_urls = [u for u in url_check if not u['valid']]

if broken_urls:
    print(f"Found {len(broken_urls)} broken URLs:")
    for url in broken_urls:
        print(f"  - {url['url']} ({url['source']})")

# Then, run judge to verify citation quality
judge = RecommendationJudge()
results = judge.evaluate_recommendations(
    recommendations=recommendations,
    milestone_context=milestone_context,
    child_info=child_info,
    recommendation_type="guide"
)

# Check citation quality scores
for eval in results['evaluations']:
    citation_quality = eval['evaluation']['medicalGuidelineCompliance']['citationQuality']
    if 'poor' in citation_quality.lower() or 'fabricated' in citation_quality.lower():
        print(f"Poor citations in: {eval['recommendationTitle']}")
        print(f"  Quality: {citation_quality}")
```

---

## Interpreting Results

### Understanding Scores

#### Child Safety (0-10)
- **9-10**: Excellent - Safe with appropriate precautions
- **7-8**: Good - Safe with minor improvements possible
- **4-6**: Concerning - Significant safety issues need addressing
- **0-3**: UNSAFE - Critical safety risks, do not use

#### Milestone Alignment (0-10)
- **9-10**: Perfectly aligned with milestone goals
- **7-8**: Good alignment, minor optimization possible
- **4-6**: Partially aligned, needs better focus
- **0-3**: Misaligned - Wrong skill or developmental area

#### Medical Guideline Compliance (0-10)
- **9-10**: Strongly evidence-based with quality citations
- **7-8**: Generally compliant, minor citation improvements
- **4-6**: Weak evidence or questionable practices
- **0-3**: Contradicts guidelines or fabricated citations

### Overall Assessment

#### PASS
- All criteria score 7 or higher
- No red flags identified
- Safe to use in production

**Action:** Deploy with confidence

#### CONDITIONAL PASS
- Scores 7+ but has minor concerns
- May have small issues noted

**Action:** Review concerns, consider minor tweaks, generally safe to deploy

#### FAIL
- Any criterion below 7 OR has red flag issues
- Not safe for production

**Action:** Do not deploy, fix issues before re-evaluation

### Red Flags

Red flags are critical issues that require immediate attention:

**Safety Red Flags:**
- Choking hazards for young children
- Age-inappropriate activities
- Contradicts medical conditions
- Missing supervision warnings

**Guideline Red Flags:**
- Contradicts AAP/CDC/WHO guidelines
- Fabricated citations
- Recommends contraindicated equipment (e.g., baby walkers)
- Dangerous practices

**Any red flag = automatic FAIL**

---

## Integration Workflows

### Workflow 1: Pre-Production Validation

```
1. Generate recommendations via API
2. Run judge evaluation
3. Review results:
   - PASS → Deploy to database
   - CONDITIONAL PASS → Human review → Deploy if approved
   - FAIL → Log issue, regenerate with adjusted prompt
4. Track metrics (pass rate, common failures)
```

### Workflow 2: Continuous Monitoring

```
1. Daily/Weekly: Sample N recommendations from production
2. Run batch evaluation
3. Generate report:
   - Overall pass rate
   - Common failure patterns
   - Red flag frequency
4. Alert if pass rate drops below threshold (e.g., 85%)
5. Monthly: Review trends, update prompts if needed
```

### Workflow 3: Prompt Refinement

```
1. Collect failed evaluations over time
2. Analyze common failure modes:
   - Safety issues pattern
   - Citation quality problems
   - Milestone misalignment types
3. Update system prompts to address patterns
4. Re-evaluate sample set with new prompts
5. Compare before/after metrics
6. Deploy improved prompts if metrics improve
```

---

## Customization

### Adjusting the Judge Prompt

You can modify the judge system prompt in `judge_evaluator.py` to:

- Add new evaluation criteria
- Adjust scoring thresholds
- Include domain-specific guidelines
- Add region-specific regulations

Example:
```python
# Add a new criterion
JUDGE_SYSTEM_PROMPT = """
... existing prompt ...

### 5. CULTURAL APPROPRIATENESS (Score 0-10)
Assess whether recommendations are culturally sensitive:
- Respects diverse family structures
- Avoids cultural biases
- Considers global practices
...
"""
```

### Using Different Models

```python
# Use Sonnet for faster/cheaper evaluations
judge = RecommendationJudge()
judge.model = "claude-sonnet-4-5-20250929"

# Use Opus for most rigorous evaluation (default)
judge.model = "claude-opus-4-5-20251101"
```

### Custom Summary Metrics

```python
def custom_analysis(results):
    """Add custom metrics to summary."""
    evaluations = results['evaluations']

    # Count recommendations with perfect safety scores
    perfect_safety = sum(
        1 for e in evaluations
        if e['evaluation']['childSafety']['score'] == 10
    )

    # Find most common concerns
    all_concerns = []
    for e in evaluations:
        all_concerns.extend(e['evaluation']['childSafety'].get('concerns', []))

    from collections import Counter
    common_concerns = Counter(all_concerns).most_common(5)

    return {
        'perfectSafetyCount': perfect_safety,
        'mostCommonConcerns': common_concerns
    }
```

---

## Troubleshooting

### API Key Issues
```bash
# Verify your API key is set
echo $ANTHROPIC_API_KEY

# Or pass directly to the judge
judge = RecommendationJudge(api_key="your-key-here")
```

### JSON Parsing Errors

If the judge output isn't valid JSON:

```python
try:
    results = judge.evaluate_recommendations(...)
except json.JSONDecodeError as e:
    print(f"Invalid JSON response: {e}")
    # The raw response is in the error - check for formatting issues
```

### Rate Limiting

If you hit rate limits:

```python
import time

for batch in recommendation_batches:
    results = judge.evaluate_recommendations(...)
    time.sleep(1)  # Add delay between requests
```

### Cost Management

Judge evaluations use Claude Opus (most capable but most expensive):

- **Cost per evaluation batch (5 recs)**: ~$0.05-0.10
- **Daily monitoring (50 samples)**: ~$0.50-1.00
- **Batch testing (500 recs)**: ~$5-10

To reduce costs:
- Use Sonnet for routine checks, Opus for critical evaluations
- Batch recommendations together (evaluate 5-10 at once)
- Sample strategically rather than evaluating everything

---

## Best Practices

### 1. Regular Cadence
- **Pre-launch**: Evaluate all recommendations
- **Weekly**: Sample 50-100 production recommendations
- **Monthly**: Full audit of prompt effectiveness

### 2. Action on Failures
- Never deploy FAIL recommendations
- Investigate patterns in CONDITIONAL PASS
- Update prompts based on failure trends

### 3. Track Metrics Over Time
```python
metrics_log = {
    "date": "2024-01-15",
    "pass_rate": 94.5,
    "avg_safety_score": 8.7,
    "red_flags": 2,
    "total_evaluated": 100
}
```

### 4. Human Review for Edge Cases
- Complex medical histories
- Controversial practices
- New milestone categories

### 5. Keep Judge Prompt Updated
- Review quarterly
- Incorporate new medical guidelines
- Update citation sources

---

## Example Output

```json
{
  "evaluations": [
    {
      "recommendationType": "guide",
      "recommendationTitle": "Practice Tummy Time Daily",
      "evaluation": {
        "childSafety": {
          "score": 9,
          "reasoning": "Tummy time is a safe, well-established practice...",
          "redFlags": [],
          "concerns": []
        },
        "milestoneAlignment": {
          "score": 10,
          "reasoning": "Directly supports rolling over by strengthening...",
          "concerns": []
        },
        "medicalGuidelineCompliance": {
          "score": 9,
          "reasoning": "Strongly aligned with CDC and AAP recommendations...",
          "redFlags": [],
          "citationQuality": "High quality - specific, accurate citations"
        },
        "overallQuality": {
          "score": 9,
          "reasoning": "Clear, actionable, evidence-based recommendation"
        }
      },
      "overallAssessment": "PASS",
      "overallReasoning": "Excellent recommendation with strong evidence base",
      "recommendedActions": ["No changes needed"]
    }
  ],
  "summary": {
    "totalEvaluated": 5,
    "passed": 4,
    "conditionalPass": 1,
    "failed": 0,
    "passRate": 80.0,
    "averageScores": {
      "childSafety": 8.8,
      "milestoneAlignment": 9.2,
      "medicalGuidelineCompliance": 8.6,
      "overallQuality": 8.9
    },
    "totalRedFlags": 0,
    "uniqueRedFlags": 0
  }
}
```

---

## Support & Updates

### Reporting Issues
If you encounter problems with the judge:
1. Check the evaluation reasoning for insights
2. Verify your input data format matches expected structure
3. Review the system prompt for criterion details

### Updating Guidelines
When new pediatric guidelines are released:
1. Update the JUDGE_SYSTEM_PROMPT with new sources
2. Add new safety considerations
3. Update the official URL list
4. Re-evaluate a sample set to verify behavior

### Contributing Improvements
Track improvements you'd like to see:
- Additional evaluation criteria
- Better citation verification
- Automated URL checking
- Integration with databases
