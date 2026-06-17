
  ## Labels
  - **hot_take** — A bold opinion stated confidently with no real supporting evidence or argument.
  - **analysis** — A structured point backed by stats, tactics, or historical comparison.
  - **reaction** — An immediate emotional response to a match or moment, with little to no argument.

  ### Examples
  **hot_take:** "Mbappe is overrated, he disappeared when it mattered most."
  **analysis:** "Argentina's high press in the second half forced 3 turnovers that led directly to goals."
  **reaction:** "I CANT BELIEVE WE LOST THAT MATCH OMG"

  ## Hard Edge Cases
  A post that includes a stat but uses it to decorate an opinion rather than build an argument (e.g., "Mbappe only scored 8 goals and France still lost — he's overrated.") → label as **hot_take**.
  Rule: if removing the opinion leaves evidence that still proves something, it's analysis. Otherwise, hot_take.

  ## Data Collection Plan
  - Source: r/soccer, r/worldcup (public posts)
  - Target: ~70 examples per label (210 total)
  - If a label is underrepresented, I will search specifically for that post type

  ## Evaluation Metrics
  - Accuracy: overall correctness
  - F1 per class: because class imbalance is possible and accuracy alone can be misleading
  - Confusion matrix: to see which labels are being confused

  ## Definition of Success
  Fine-tuned model achieves at least 70% overall accuracy and beats the Groq baseline by at least 10 percentage points.

  ## AI Tool Plan
  - **Label stress-testing:** Ask AI to generate 10 borderline posts to test label definitions
  - **Annotation assistance:** Use AI to pre-label batches, then review and correct each one myself
  - **Failure analysis:** After training, paste wrong predictions into AI and ask it to find patterns
