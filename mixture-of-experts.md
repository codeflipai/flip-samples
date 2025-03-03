// Define expert functions - each expert is specialized for a different task
fun language-expert input: 
  input >> `Analyze this text for grammar and style improvements.`

fun sentiment-expert input: 
  input >> `Analyze the sentiment in this text (positive, negative, or neutral).`

fun factual-expert input: 
  input >> `Check this text for factual accuracy and provide corrections if needed.`

fun summarization-expert input: 
  input >> `Summarize the key points from this text in bullet points.`

// Task Manager - decomposes the main task into subtasks with explicit task types
fun task-manager input:
  input >> `Break down this task into subtasks focused on these specific aspects:
  - Language analysis
  - Sentiment analysis
  - Factual verification
  - Summarization
  
  Return a JSON object with fields:
  - language_task: text for language analysis
  - sentiment_task: text for sentiment analysis
  - factual_task: text for factual verification
  - summary_task: text for summarization` >> object()

// Process all subtasks using appropriate experts
fun process-subtasks task-object:
  {
    language_result: task-object.language_task >> language-expert,
    sentiment_result: task-object.sentiment_task >> sentiment-expert,
    factual_result: task-object.factual_task >> factual-expert,
    summary_result: task-object.summary_task >> summarization-expert
  }

// Aggregator - combines expert outputs into a final result
fun aggregator results original-task:
  {
    task: original-task,
    expert_results: results
  } >> `You are a final aggregator. Based on these expert analyses, 
  create a unified response that addresses the original task.`

// Main MoE function
fun mixture-of-experts task:
  let([
    // 1. Break down the task into specific subtasks
    subtasks: task-manager(task),
    
    // 2. Process all subtasks with their respective experts
    results: process-subtasks(subtasks),
    
    // 3. Combine the results
    final-output: aggregator(results, task)
  ], final-output)

// Example with realistic input
main:
  "Analyze how climate change affects global food security, focusing on wheat, rice, and maize production. Consider temperature changes, water availability, and extreme weather events."
  >> mixture-of-experts