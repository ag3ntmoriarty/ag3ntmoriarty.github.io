---
title: "From Random Moves to Forced Mates: Theoretical Lessons from Building an RL Chess Agent"
date: 2026-10-07
tags:
  - Reinforcement_Learning
  - MCTS
---
Building an AlphaZero-inspired chess engine from scratch is an exercise in balancing intuition (neural networks) with calculation (tree search). Over the course of developing this agent, I observed several fascinating theoretical dynamics regarding how machines learn, evaluate, and navigate the massive state-space of chess.

Here are the key theoretical observations and conclusions drawn from the architecture, reinforcement learning dynamics, and search optimizations of the project.

### 1. Spatial Intuition: Why Transformers Outperform Convolutions

Initially, the agent was built using standard 2D Convolutional Neural Networks (CNNs) to process the 8x8 chessboard. However, chess is a game of both local tactics and long-range threats. A bishop on `a1` pinning a knight on `f6` to a king on `h8` spans the entire board.

**Observation:** Convolutions naturally struggle with these long-range dependencies because their receptive fields are localized. To connect `a1` to `h8`, a CNN requires many deep layers, making it parameter-heavy and slow to train.

**Conclusion:** Treating the chessboard not as an image, but as a sequence of 64 discrete tokens allows for a **Vision Transformer (ViT)** architecture. By utilizing self-attention mechanisms, a Transformer can immediately evaluate the relationship between any two squares on the board in a single layer. Shifting to a Transformer backbone unified the policy and value heads effectively and gave the agent an immediate sense of global board tension that convolutions lacked.

### 2. The "Tabula Rasa" Myth: Bootstrapping the Cold Start

The original AlphaZero famously learned entirely from self-play, starting from random weights (Tabula Rasa). However, doing this requires Google-scale compute clusters.

**Observation:** When a randomly initialized agent plays against itself, the games devolve into random piece shuffling. The reward signal in chess is incredibly sparse—you only get a `+1` or `-1` after dozens of moves. A locally trained agent might play thousands of games without ever stumbling into a checkmate, meaning it receives zero gradient updates telling it *how* to win.

**Conclusion:** For practical RL applications, pure self-play is computationally prohibitive. Bootstrapping the network using supervised learning on a dataset of Grandmaster games (like KingBase) is essential. It acts as a behavioral primer. Once the network learns the basic grammar of chess (opening principles, king safety, development), it can transition into RL self-play to fine-tune its evaluations and discover novel strategies.

### 3. Search Space Amnesia and MCTS Stability

Monte Carlo Tree Search (MCTS) is brilliant at evaluating deep tactical sequences by simulating future rollouts. But search trees are highly vulnerable to cyclical logic.

**Observation:** In early iterations, the MCTS would fall into infinite move-repetition loops. It would find a threatening move, the opponent would block, it would retreat, and then play the exact same threat. The Expected Value (EV) would wildly fluctuate between highly winning and highly losing for the exact same board positions.

**Conclusion:** A Markov Decision Process (MDP) assumes the current state contains all necessary information. But chess is *not* strictly Markovian if you only look at the pieces—rules like threefold repetition and the 50-move rule require historical context.

If MCTS suffers from "history amnesia," it treats cyclic retreats as fresh opportunities rather than forced draws. Furthermore, low simulation budgets exacerbate EV variance, preventing the agent from distinguishing a shallow, one-move threat from a deep, structured plan. MCTS requires strict historical state-tracking and high simulation depth to stabilize its value predictions.

### 4. The Endgame Phase Shift: Intuition vs. Calculation

Neural networks are excellent at evaluating complex, crowded middlegames. They rely on pattern recognition (intuition).

**Observation:** As pieces leave the board, the state space becomes sparse. Ironically, the neural network struggles immensely in simple endgames (e.g., King and Rook vs. King). It recognizes the position is "winning" (high value output), but it aimlessly shuffles the rook around because it lacks the precise, deterministic calculation required to force a mate in 15 moves.

**Conclusion:** Chess requires a phase shift in computation. While neural networks dominate the fuzzy logic of the middlegame, sparse endgames require clinical, symbolic search.

Switching from neural-guided MCTS to a deterministic **Negamax (Alpha-Beta) solver** when the piece count drops below a certain threshold drastically improves endgame conversion.

Furthermore, two critical optimizations are required for mathematical solvers:

1. **Transposition Tables (Zobrist Hashing):** The endgame tree is incredibly wide. Caching previously evaluated positions is mandatory to achieve the depth required to spot mates.

2. **Mate-Distance Scoring:** If mate-in-2 and mate-in-20 both return an evaluation of `+Infinity`, the agent will meander. By scoring mates as `Infinity - depth`, the algorithm inherently gravitates toward the most ruthlessly efficient path to victory.

### 5. Shaping the Reward Landscape and Action Spaces

In RL, the agent optimizes strictly for the environment's reward. If the only reward is at the end of the game, learning stalls.

**Observation:** During early self-play, the agent would occasionally ignore hanging pieces or refuse to promote pawns that had reached the 7th rank.

**Conclusion:** **Reward shaping** acts as a catalyst for RL. By injecting dense, intermediate heuristics—such as granting a micro-reward (`+0.1`) for capturing an undefended piece, or a massive bonus (`+0.2`) for pawn promotions—the agent is guided toward tactically sound principles much faster than waiting for terminal rewards.

Additionally, the **action space representation** must be expressive enough to capture all nuances. If all pawn promotions (Queen, Rook, Knight, Bishop) are mapped to a single abstract "promotion" action index, the neural network loses the ability to differentiate them. High-fidelity action mapping is required for the policy network to learn edge cases like underpromotions.

---
#### After all the changes and trial and errors. Here is a sample game between 2 agents trained at different number of episodes.

![[rlchesssamplegame.gif]]

### Final Thoughts
Building this engine highlighted the beautiful duality of modern AI. The most effective systems don't rely purely on modern deep learning or purely on classical algorithms. They combine the broad, pattern-matching intuition of Neural Networks with the sharp, exhaustive precision of traditional search algorithms.