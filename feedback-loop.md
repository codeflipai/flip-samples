## AI Feedback Loops

### Description

The AI Feedback Loops pattern involves multiple AI components interacting in a cyclical manner. The output of one AI serves as the input for another, creating a continuous loop that refines results iteratively. This pattern enhances system performance through self-correction and adaptation based on feedback.

### Components

-  **Primary AI**: Generates initial outputs.
-  **Secondary AI**: Evaluates, critiques, or enhances the outputs of the Primary AI.
-  **Feedback Mechanism**: Facilitates communication between AIs for iterative improvement.

### Workflow

-  **Initial Output**: The Primary AI produces an output based on input data.
-  **Evaluation**: The Secondary AI analyzes the output for quality, accuracy, or compliance.
-  **Feedback Integration**: The Primary AI receives feedback and adjusts its processing accordingly.
-  **Iteration**: Steps 1–3 repeat until the desired quality is achieved.

### Potential Use Cases

-  **Refined Language Translation**: One AI translates text; another reviews for context and idiomatic expressions, enhancing the translation iteratively.
-  **Automated Coding**: An AI writes code based on requirements; a secondary AI reviews for bugs or inefficiencies, leading to improved code quality.
-  **Medical Diagnosis**: An AI suggests a diagnosis; another AI cross-checks with medical databases to confirm or refine the diagnosis.

### Example Implementation

```flip

// Primary AI that generates content based on a prompt
fun generate-content prompt revision_context=null:
  if(revision_context >> is-null?(),
    // Initial content generation - pass prompt directly
    prompt >> `Create content based on this prompt`,

    // Revision based on feedback - pass context object
    {
      prompt: prompt,
      previous_content: revision_context.previous_content,
      feedback: revision_context.feedback
    } >> `Revise the content based on the provided prompt, previous content, and feedback`
  )

// Secondary AI that evaluates content for quality and criteria
fun evaluate-content content criteria:
  {
    content: content,
    criteria: criteria
  } >> `Evaluate this content based on the provided criteria. For each criterion, provide a score (1-10), specific feedback, and whether it meets the criterion (true/false). Return as structured JSON.` >> object()

// Function to check if all criteria are met
fun all-criteria-met evaluation:
  evaluation.criteria >> map(criterion -> criterion.meets_criteria) >> every(x -> x == true)

// Function to extract actionable feedback from evaluation
fun extract-feedback evaluation:
  evaluation.criteria >>
    filter(criterion -> criterion.meets_criteria == false) >>
    map(criterion -> criterion.feedback) >>
    join("\n\n")

// Main feedback loop function
fun content-feedback-loop prompt criteria max_iterations=5:
  let([
    // Initialize loop state
    initial_state: {
      content: prompt >> generate-content,
      iteration: 1,
      history: [],
      criteria: criteria
    }
  ],
    // Start the iterative loop
    loop(initial_state, state ->
      let([
        // Evaluate current content
        evaluation: evaluate-content(state.content, state.criteria),

        // Check if all criteria are met or max iterations reached
        criteria_met: all-criteria-met(evaluation),
        max_reached: state.iteration >= max_iterations,

        // Record this iteration in history
        new_history: state.history >> concat([{
          iteration: state.iteration,
          content: state.content,
          evaluation: evaluation
        }]),

        // Determine next steps based on conditions
        next_action: match(
          criteria_met => {
            status: "success",
            message: "All criteria met successfully",
            final_content: state.content,
            iterations: state.iteration,
            history: new_history
          },

          max_reached => {
            status: "max_iterations",
            message: "Maximum iterations reached before all criteria met",
            final_content: state.content,
            iterations: state.iteration,
            history: new_history
          },

          _ => {
            // Extract feedback and create revision context
            feedback: extract-feedback(evaluation),
            revision_context: {
              previous_content: state.content,
              feedback: feedback
            },

            // Generate improved content based on feedback
            improved_content: generate-content(prompt, revision_context),

            // Continue the loop with updated state
            next_state: {
              content: improved_content,
              iteration: state.iteration + 1,
              history: new_history,
              criteria: state.criteria
            }
          }
        )
      ],
        // If we have a result, return it; otherwise recur with the next state
        match(
          next_action.status >> is-defined?() => next_action,
          _ => recur(next_action.next_state)
        )
      )
    )
  )

// Define evaluation criteria for different content types
fun blog-post-criteria:
  [
    {name: "clarity", description: "Content is clear and easy to understand"},
    {name: "factual_accuracy", description: "All facts and claims are accurate"},
    {name: "engagement", description: "Content is engaging and maintains reader interest"},
    {name: "seo_optimization", description: "Content includes relevant keywords and is optimized for search"},
    {name: "call_to_action", description: "Content includes effective calls to action"}
  ]

fun marketing-copy-criteria:
  [
    {name: "persuasiveness", description: "Copy is persuasive and compelling"},
    {name: "brand_voice", description: "Copy matches the brand's tone and voice"},
    {name: "benefit_focus", description: "Copy emphasizes customer benefits rather than features"},
    {name: "conciseness", description: "Copy is concise and impactful"},
    {name: "call_to_action", description: "Copy includes clear and effective calls to action"}
  ]

// Example usage - create a blog post with feedback loops
main prompt:
  let([
    // Determine content type and criteria based on prompt
    content_type: prompt >> `Analyze this prompt and determine if it's requesting a blog post or marketing copy. Return only "blog_post" or "marketing_copy"`,

    criteria: match(
      content_type == "blog_post" => blog-post-criteria(),
      content_type == "marketing_copy" => marketing-copy-criteria(),
      _ => blog-post-criteria() // default to blog post criteria
    ),

    // Run the feedback loop
    result: content-feedback-loop(prompt, criteria),

    // Generate a summary of the improvement process
    improvement_summary: result >> `Analyze the content improvement across iterations. Highlight key changes made and how the feedback was incorporated.`
  ],
    // Return the final result with summary
    {
      final_content: result.final_content,
      iterations_required: result.iterations,
      improvement_summary: improvement_summary,
      met_all_criteria: result.status == "success"
    }
  )
```
