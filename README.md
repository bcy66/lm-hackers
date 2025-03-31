# A Hackers' Guide to Language Models

This material is to accompany the video [A Hackers' Guide to Language Models](https://youtu.be/jkrNMKz9pWU). The notebook from the video is [lm-hackers.ipynb](lm-hackers.ipynb). The other files are for axolotl fine-tuning.


# LLM Prompt Templates for SVA Evaluation

readability_prompt: |
  <instructions>
  As a senior design verification engineer, evaluate the readability of the following SystemVerilog assertions code.
  
  You're analyzing code from module '{module_name}', focusing on a group of related assertions.
  </instructions>
  
  <code>
  {code_chunk}
  </code>
  
  <evaluation_criteria>
  Please evaluate the code on the following readability aspects:
  1. Clarity of intention - Is the purpose of each assertion clear?
  2. Appropriate comments - Are there helpful comments explaining complex assertions?
  3. Consistent formatting - Is indentation and spacing consistent?
  4. Understandable property expressions - Are temporal expressions easy to understand?
  5. Clear signal naming - Do signal names clearly indicate their purpose?
  
  Rate each criterion from 0.0 (poor) to 1.0 (excellent).
  </evaluation_criteria>
  
  <example>
  Consider this example that has poor readability:
  
  ```systemverilog
  property p1;
    @(posedge clk) a |=> b ##1 c;
  endproperty
  assert property(p1);
  ```
  
  Improved version:
  
  ```systemverilog
  // Verify that signal b asserts one cycle after a, followed by c
  property ap_a_followed_by_b_then_c;
    @(posedge clk) a |=> b ##1 c;
  endproperty
  assert property(ap_a_followed_by_b_then_c);
  ```
  </example>
  
  <format>
  Provide your evaluation in this format:
  
  Overall Readability Score: [0.0-1.0]
  
  Scores by Criterion:
  - Clarity of intention: [0.0-1.0]
  - Appropriate comments: [0.0-1.0]
  - Consistent formatting: [0.0-1.0]
  - Understandable property expressions: [0.0-1.0]
  - Clear signal naming: [0.0-1.0]
  
  Issues:
  1. [Issue description] (Line: X, Severity: [LOW|MEDIUM|HIGH|CRITICAL])
     Suggestion: [Improvement suggestion]
  2. ...
  
  Summary: [Brief summary of evaluation with key improvements needed]
  </format>

completeness_prompt: |
  <instructions>
  As a senior design verification engineer, evaluate the completeness of the following SystemVerilog assertions code.
  
  You're analyzing code from module '{module_name}'. Your goal is to identify potential verification gaps or missing assertions based on the functionality implied by the RTL and existing assertions.
  </instructions>
  
  <module_io>
  {module_io}
  </module_io>
  
  <existing_assertions>
  {existing_assertions}
  </existing_assertions>
  
  <rtl_snippets>
  {rtl_snippets}
  </rtl_snippets>
  
  <evaluation_criteria>
  Look for these common verification gaps:
  1. Reset behavior - Are all signals properly verified during/after reset?
  2. Protocol compliance - Are all protocol phases and requirements verified?
  3. State transitions - Are all valid and invalid state transitions covered?
  4. Error conditions - Are error handling and recovery verified?
  5. Timing requirements - Are all timing constraints verified?
  6. Signal interactions - Are relationships between signals properly verified?
  7. Corner cases - Are boundary conditions checked?
  8. Exclusivity - Are mutually exclusive conditions verified?
  
  For each gap, suggest specific assertions that should be added.
  </evaluation_criteria>
  
  <example>
  Given an AXI interface with assertions for VALID/READY handshaking, you might identify:
  
  Missing assertion: No verification that VALID doesn't deassert before READY
  Suggested property:
  ```systemverilog
  property ap_valid_stable_until_ready;
    @(posedge clk) (s_valid && !s_ready) |=> s_valid;
  endproperty
  ```
  </example>
  
  <format>
  Provide your evaluation in this format:
  
  Completeness Score: [0.0-1.0]
  
  Verification Gaps:
  1. [Gap description] (Severity: [LOW|MEDIUM|HIGH|CRITICAL])
     Missing assertion: [Brief description of what's not verified]
     Suggested property:
     ```systemverilog
     [Code for suggested assertion]
     ```
  2. ...
  
  Summary: [Brief summary of completeness evaluation]
  </format>

reusability_prompt: |
  <instructions>
  As a senior design verification engineer, evaluate the reusability and modularity of the following SystemVerilog assertions code.
  
  You're analyzing code from module '{module_name}', focusing on how well-structured and reusable the assertions are.
  </instructions>
  
  <code>
  {code_chunk}
  </code>
  
  <evaluation_criteria>
  Please evaluate the code on the following reusability aspects:
  1. Parameterization - Are parameters used effectively?
  2. Abstraction - Are sequences and properties defined separately from assertions?
  3. Modularity - Are related properties grouped logically?
  4. Generalization - Are assertions written to be reusable across similar designs?
  5. Hardcoded values - Are magic numbers avoided in favor of parameters?
  
  Rate each criterion from 0.0 (poor) to 1.0 (excellent).
  </evaluation_criteria>
  
  <example>
  Non-reusable assertion:
  ```systemverilog
  assert property(@(posedge clk) req |-> ##[1:5] ack);
  ```
  
  More reusable version:
  ```systemverilog
  parameter MAX_ACK_LATENCY = 5;
  
  sequence req_to_ack_seq(req, ack, max_latency);
    req |-> ##[1:max_latency] ack;
  endsequence
  
  property ap_req_to_ack;
    @(posedge clk) req_to_ack_seq(req, ack, MAX_ACK_LATENCY);
  endproperty
  assert property(ap_req_to_ack);
  ```
  </example>
  
  <format>
  Provide your evaluation in this format:
  
  Overall Reusability Score: [0.0-1.0]
  
  Scores by Criterion:
  - Parameterization: [0.0-1.0]
  - Abstraction: [0.0-1.0]
  - Modularity: [0.0-1.0]
  - Generalization: [0.0-1.0]
  - Avoidance of hardcoded values: [0.0-1.0]
  
  Issues:
  1. [Issue description] (Line: X, Severity: [LOW|MEDIUM|HIGH|CRITICAL])
     Suggestion: [Improvement suggestion]
  2. ...
  
  Summary: [Brief summary of evaluation with key improvements needed]
  </format>

efficiency_prompt: |
  <instructions>
  As a senior design verification engineer, evaluate the efficiency and performance impact of the following SystemVerilog assertions code.
  
  You're analyzing code from module '{module_name}', focusing on potential simulation performance impacts.
  </instructions>
  
  <code>
  {code_chunk}
  </code>
  
  <evaluation_criteria>
  Please evaluate the code on the following efficiency aspects:
  1. Complexity of temporal expressions - Are assertions unnecessarily complex?
  2. Unbounded ranges - Are there unbounded or very large ranges?
  3. Execution frequency - How often will the assertion be evaluated?
  4. Redundancy - Are there redundant assertions checking similar properties?
  5. Resource usage - Will assertions consume significant simulator resources?
  
  Rate each criterion from 0.0 (poor) to 1.0 (excellent).
  </evaluation_criteria>
  
  <example>
  Inefficient assertion:
  ```systemverilog
  property ap_eventual_ack;
    @(posedge clk) req |-> ##[1:$] ack;
  endproperty
  assert property(ap_eventual_ack);
  ```
  
  More efficient version:
  ```systemverilog
  property ap_eventual_ack;
    @(posedge clk) req |-> ##[1:MAX_ACK_LATENCY] ack;
  endproperty
  assert property(ap_eventual_ack);
  ```
  </example>
  
  <format>
  Provide your evaluation in this format:
  
  Overall Efficiency Score: [0.0-1.0]
  
  Scores by Criterion:
  - Complexity of temporal expressions: [0.0-1.0]
  - Use of bounded ranges: [0.0-1.0]
  - Execution frequency considerations: [0.0-1.0]
  - Avoidance of redundancy: [0.0-1.0]
  - Resource usage: [0.0-1.0]
  
  Issues:
  1. [Issue description] (Line: X, Severity: [LOW|MEDIUM|HIGH|CRITICAL])
     Suggestion: [Improvement suggestion]
  2. ...
  
  Summary: [Brief summary of evaluation with key improvements needed]
  </format>

complexity_prompt: |
  <instructions>
  As a senior design verification engineer, evaluate the complexity of the following SystemVerilog assertions code.
  
  You're analyzing code from module '{module_name}', focusing on cognitive complexity.
  </instructions>
  
  <code>
  {code_chunk}
  </code>
  
  <evaluation_criteria>
  Please evaluate the code on the following complexity aspects:
  1. Temporal depth - How many nested temporal operators are used?
  2. Signal count - How many signals are referenced in each property?
  3. Logical complexity - How complex are the logical expressions?
  4. Readability - Is the assertion structure easy to comprehend?
  5. Decomposition - Are complex properties broken down appropriately?
  
  Rate each criterion from 0.0 (poor) to 1.0 (excellent).
  </evaluation_criteria>
  
  <example>
  Overly complex assertion:
  ```systemverilog
  property ap_complex;
    @(posedge clk) 
    (a && b && c) |-> 
    ##1 (d || e) && ##[2:5] (f && g) |-> ##1 (h || i) ##2 j;
  endproperty
  assert property(ap_complex);
  ```
  
  Better decomposed version:
  ```systemverilog
  sequence seq_initial_condition;
    a && b && c;
  endsequence
  
  sequence seq_middle_stage;
    (d || e) && ##[2:5] (f && g);
  endsequence
  
  sequence seq_final_stage;
    (h || i) ##2 j;
  endsequence
  
  property ap_decomposed;
    @(posedge clk) 
    seq_initial_condition |-> 
    ##1 seq_middle_stage |-> ##1 seq_final_stage;
  endproperty
  assert property(ap_decomposed);
  ```
  </example>
  
  <format>
  Provide your evaluation in this format:
  
  Overall Complexity Score: [0.0-1.0] (higher is better - means less complex)
  
  Scores by Criterion:
  - Temporal depth: [0.0-1.0]
  - Signal count: [0.0-1.0]
  - Logical complexity: [0.0-1.0]
  - Readability: [0.0-1.0]
  - Decomposition: [0.0-1.0]
  
  Issues:
  1. [Issue description] (Line: X, Severity: [LOW|MEDIUM|HIGH|CRITICAL])
     Suggestion: [Improvement suggestion]
  2. ...
  
  Summary: [Brief summary of evaluation with key improvements needed]
  </format>