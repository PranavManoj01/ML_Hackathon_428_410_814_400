# ML_Hackathon_428_410_814_400

## Hangman AI Agent (HMM + Q-Learning)

This project is an intelligent Hangman agent built for the ML HackMan challenge. It uses a hybrid approach, combining a statistical Hidden Markov Model (HMM) with an experience-based Reinforcement Learning (RL) agent to efficiently solve Hangman puzzles.

The HMM acts as a "book smart" oracle, providing statistical probabilities for letters. The RL agent (Q-Learning) acts as an "experienced player," learning a policy from playing thousands of games. The final agent blends these two "brains" to make an optimal guess.

## Final Performance (on 2,000-word Test Set)

| Metric | Result |
| :--- | :--- |
| **Success Rate** | 32.55% |
| **Average Wrong Guesses** | 5.22 |
| **Total Repeated Guesses** | 0 |
| **Final Score** | -51,594.00 |

*A key insight from the evaluation is that this agent performs significantly better on longer words (11+ letters), achieving 88% on 16-letter words and 100% on 18-letter words. This is likely due to the HMM's bigram model having more context to work with.*

## System Architecture

The agent's decision-making process is a hybrid of two components:

### 1. The "Oracle": Hidden Markov Model (HMM)

This component, defined in the `HangmanHMM` class, provides statistical "book smarts."

* **Model:** It uses a **Bigram Model**, learning the probability of a letter appearing given the *previous* letter (e.g., `P('H' | 'T')`).
* **Training:** It trains 24 separate models, one for each word length found in the 50,000-word corpus (from 1 to 24 letters).
* **Prediction:** At any point, it can provide a list of the most probable next letters based on the current masked word and the corpus statistics.

### 2. The "Learner": Reinforcement Learning (Q-Learning)

This component, defined in the `HangmanRLAgent` class (in Cell 6), learns from experience.

* **Algorithm:** Tabular Q-Learning.
* **Q-Table (The "Cheat Sheet"):** It builds a large dictionary (`q_table`) that stores the expected future reward (the Q-value) for taking any action in any given state.
* **State (`get_state_key`):** The state is a simple, hashable string: `"{masked_word}|{lives_remaining}|{guess_count}"`.
    * *Example:* `"_ P P _ _ | 5 | 3"`
* **Reward Function:** The agent is trained using the following rewards from the `HangmanEnvironment`:
    * **Win:** +100
    * **Correct Guess:** +20
    * **Wrong Guess:** -10
    * **Loss:** -100
    * **Repeated Guess:** -2

### 3. The "Hybrid Brain": Decision Blending

This is the "secret sauce" of the agent (`_get_best_action` function). When it's time to make a guess, the agent:

1.  Gets the statistical scores from the **HMM (Oracle)**.
2.  Gets the learned experience scores from its **Q-Table (Learner)**.
3.  Blends them using a weighted average:
    **`Final Score = (60% * HMM Probability) + (40% * Q-Value)`**

It then picks the letter with the highest combined score.

## How to Run

This project is contained within a single Jupyter/Colab notebook. The cells are designed to be run in order.

1.  **Cell 1:** Run to import all necessary packages.
2.  **Cell 2:** Run to upload your `corpus.txt` and `test.txt` files.
3.  **Cell 3:** Run to perform the initial analysis of the `corpus.txt` file and generate plots.
4.  **Cell 4:** Run to define the `HangmanHMM` and `HangmanEnvironment` classes.
5.  **Cell 5:** (*Optional*) This cell defines an alternative "pure" RL agent that is **not used** in the final solution. You can skip running it.
6.  **Cell 6:** **Run this cell.** This defines the actual `HangmanRLAgent` that uses the 60/40 hybrid strategy.
7.  **Cell 7 (Main Training):** Run this to train the agent for 25,000 episodes. This is the main training step and will take some time. The training log will be printed as it learns.
8.  **Cell 8:** Run to visualize the training performance (Success Rate vs. Episodes, etc.).
9.  **Cell 9 (Evaluation):** Run this to evaluate the trained agent against the 2,000-word `test.txt`. This will print the final score and performance by word length.
10. **Cell 10:** Run to visualize the detailed evaluation results.
11. **Cell 11:** Run to play an interactive demo game against the trained agent.
12. **Cell 12:** Run to generate the final Markdown-formatted analysis report.

## File Structure
├── ML_HackMan.ipynb # The main Jupyter notebook with all code 
└── Analysis_report.pdf

## Future Improvements

(Based on the analysis in the notebook)
* **DQN:** Replace the tabular Q-Learning with a Deep Q-Network (DQN) to handle a much more complex state (like the HMM probability vector) and improve generalization.
* **Richer HMM:** Enhance the HMM to a trigram model (`P(letter | prev_two_letters)`) for even better statistical accuracy.
* **Curriculum Learning:** Train the agent on easy (short) words first, then gradually increase the difficulty to longer words to help it learn more effectively.