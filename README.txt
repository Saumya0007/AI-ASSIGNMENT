==========================================================================
 AI Assignment - Classical Reasoning Agent
==========================================================================

WHAT THIS PROJECT DOES
Given a knowledge base (facts + rules, or formulas in logic) and a statement,
the system decides whether the statement is entailed (True) or not (False).
The decision is always made by a classical reasoning engine. No generative AI
is used at any stage.

Pipeline:   input  ->  representation  ->  classical reasoning  ->  True / False

--------------------------------------------------------------------------
FOLDER CONTENTS
--------------------------------------------------------------------------
  reasoning_agent.ipynb   complete implementation (run all cells top to bottom)
  report.pdf              development journal: representation, algorithms,
                          experiments, reasoning traces, limitations
  submission.csv          predictions (columns: id,label) - produced by the last
                          notebook cell from the hidden test file
  README.txt              this file

--------------------------------------------------------------------------
HOW TO RUN
--------------------------------------------------------------------------
1. Requirements: Python 3.10+, pandas, pyarrow, jupyter
       pip install pandas pyarrow jupyter
2. Put the public data next to the notebook:
       datasets/ruletaker/{train,test,ood_test}.parquet
       datasets/FOLIO/folio_v2_{train,validation}.jsonl
       datasets/counterbench/data_balanced_alpha_V1.json
       datasets/counterbench/data_balanced_backdoor_V2.json
3. Open reasoning_agent.ipynb and choose "Run All".
   Cells must run in order, because later cells use functions defined earlier.

NOTEBOOK CELLS
  CELL 1   imports, dataset paths, name of the hidden test file
  CELL 2   load and inspect the datasets
  CELL 3   rule engine: parser, forward chaining, backward chaining,
           counterfactual solver (original engine)
  CELL 4   backward-chaining proof search as state-space search:
           BFS, DFS, UCS, A*
  CELL 5   first-order logic: tolerant parser (Unicode and ASCII notation),
           CNF + Skolemisation, unification, resolution (clause selection by
           BFS / DFS / A*), DPLL
  CELL 6   structural-causal-model solver (CounterBench)
  CELL 7   predict() - the router - and evaluation on all public data;
           prints accuracy and inference time per dataset and overall
  CELL 8   comparison of the search strategies
  CELL 9   example reasoning traces
  CELL 10  reads the hidden test file and writes submission/submission.csv for random public 250 datasets
  CELL 11  reads the hidden test file and writes submission/submission.csv

--------------------------------------------------------------------------
HIDDEN TEST SET -> submission.csv
--------------------------------------------------------------------------
Put the private file next to the notebook and set  v_HIDDEN_EVAL_FILE
(in CELL  SINGLE SELF-CONTAINED CELL; default "private_test.csv"). csv, json, jsonl and parquet work.
Column names are detected automatically:
   natural language : context | premises | given_info   and   statement | question
   logic            : premises-FOL | premises_fol | ...  and   conclusion-FOL | conclusion_fol | ...
   optional         : id   (the row index is used if there is no id column)
If the file is not found, CELL 11 falls back to the public RuleTaker test split
so that the output format can be checked; use the real private file for the
final submission.csv.
Output: submission/submission.csv with columns  id,label   (label = True or False).

--------------------------------------------------------------------------
HOW THE MODEL CHOOSES A REASONING PATH
--------------------------------------------------------------------------
predict() never sees a dataset name; it looks at the FORMAT of the input:
  1. logic supplied as separate columns            -> FOL prover
  2. logic notation inside the context text        -> FOL prover
     (Unicode: forall exists -> & v ~ ... or ASCII: forall, exists, ->, &, |, ~)
  3. text containing a Bernoulli prior  Bern(p)    -> causal-model solver
  4. anything else (rules in English)              -> parser + forward chaining
If any step raises an error, the prediction is False (the run never crashes).

--------------------------------------------------------------------------
METHODS USED
--------------------------------------------------------------------------
  Forward chaining ............ default for rule bases (with proof trees)
  Backward chaining ........... alternative engine + state-space search
  BFS / DFS / UCS / A* ........ proof-search frontier, and clause selection in resolution
  Unification (occurs check) .. rule engine and resolution
  Resolution .................. first-order refutation (factoring, tautology removal)
  DPLL ........................ ground SAT: unit propagation, pure literals, branching
  CNF + Skolemisation ......... preparing first-order formulas for the provers
  Stratified negation ......... negation as failure in the rule engine
  Model enumeration ........... abduction-action-prediction for counterfactuals

