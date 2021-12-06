---
layout: post
title: Minesweeper solver using NN
---

_Alisher Tortay, Oleg Yurchecnko_  
KAIST, 2016

## Abstract

[MineSweeper](https://en.wikipedia.org/wiki/Minesweeper_(video_game)) is a popular puzzle game developed by Microsoft. For the past two decades a number of approaches to solve the game were presented. Most of them are based on estimating the probability of a covered tile to be a mine. Recent research from [Stanford University](http://cs229.stanford.edu/proj2015/372_report.pdf) proposed an interesting attempt to apply modern reinforcement learning techniques to this problem. In this paper we present another approach for solving MineSweeper game using neural network classifier. We were able to obtain results comparable to those of  the state of the art algorithms.

You can find [source code with original paper](https://github.com/Chizhik/minesweeper) and [sample video](https://youtu.be/m93DKCp-P0c) on github and youtube, respectively.

## Why Minesweeper?

Playing this game requires use of logic and awareness of relations between neighboring fields. It is a game of imperfect information and checking the state of board for solutions is [NP complete problem](http://web.mat.bham.ac.uk/R.W.Kaye/minesw/ordmsw.htm). Therefore, playing minesweeper with neural networks is challenging and interesting research topic in Artificial Intelligence. Our goal in this project was to implement Neural Network based agent that can play Minesweeper.

## Literature Review

Recent development of machine learning caused a revival in interest in using machine learning and reinforcement learning techniques in the computer game industry. During the past few years number of different attempts have been made to solve wide range of games. State of the art algorithms for RL include Q-learning, policy gradient, and number of other approaches. For instance, in 2013 DeepMind in their [paper](https://deepmind.com/research/publications/playing-atari-deep-reinforcement-learning/) presented a deep convolutional neural network using which they achieved huge performance in playing Atari 2600 games. Similar networks and techniques were used for other games, such as Flappy Bird, Out Run, etc.

As for MineSweeper solver, wide range of methods had been applied starting from analytical solutions, genetic algorithms to Q-learning method. The first attempts to solve MineSweeper game were developed in 1990’s. Such that, in 1997, A. Adamatzky showed how to mark all mines populated on n*n board using two-dimensional cellular automaton in (n)time. The state of art method for solving MineSweeper game is limited search with probability estimates strategy proposed by [K. Pedersen in 2004](http://www.minesweeper.info/articles/ComplexityMinesweeperStrategiesForGamePlaying.pdf). The solver based on this method achieved 92,5% win rate on the beginner board, and  25% on expert level. 

There also have been few attempts to use machine to play the game. In 2015 [students from Stanford University](http://cs229.stanford.edu/proj2015/372_report.pdf) applied supervised learning using SVM classifier and linear regression model, and Q-learning algorithm based on Q-tables. However, performance of those approaches dramatically decreases as the size of the board increases. For instance, the win-rate for Q-learning algorithm reaches 70% for 4*4 board with 3 mines, but falls to the level of around 5% for 5*5 boards with 5 mines (the density of mines is nearly 20% on those boards). Meanwhile, other algorithms have even lower win rate.

## Methods and Approach

Due to the lack of availability of Minesweeper frameworks capable of implementing an automated player, we wrote our own version of minesweeper game. In our game there is no mine at position (0,0). Interface works as follows: we open a square using open(pos) function and mark a tile using mark(pos) function. This functions returns state in the form of array(1,n*m) and whether game is finished. From the game we also can obtain true positions of all mines; we will use this information to train our agents.

### Supervised Learning Agent (SLA)

We used supervised learning model to implement this agent. We designed three feed-forward neural networks with different sizes. Input is subboard of size `5x5`. A tile has 12 possible values: uncovered, flag, out-of-board, and integers from 0 to 8. Each tile in this subboard is represented using one hot encoding. Output is one number between 0 and 1: `probability that central tile of the 5x5 subboard does not contain a mine`. We trained our networks with with following values: 0 if central tile of 5x5 input contains a mine and 1 otherwise. We define perimeter as a set of tiles that have at least one uncovered or marked neighbor.

#### Supervised Learning Algorithm
1.  begin by probing upper-left corner
2.  **while** not game over **do**
3.  &nbsp;&nbsp;&nbsp;&nbsp;**foreach** tile in the perimeter
4.  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;predict probability
5.  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**if** probability < 0.1
6.  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mark the tile as mine
7.  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**else if** probability > 0.1
8.  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;unmark the tile
9.  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**end if**
10. &nbsp;&nbsp;&nbsp;&nbsp;**end foreach**
11. &nbsp;&nbsp;&nbsp;&nbsp;**foreach** tile in the perimeter
12. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;predict probability
13. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;save this training sample into experience buffer
14. &nbsp;&nbsp;&nbsp;&nbsp;**end foreach**
15. &nbsp;&nbsp;&nbsp;&nbsp;open a tile with maximal probability
16. **end while**
17. train network using random sample from experience buffer

As you can see from the algorithm we save training samples into experience buffer and trained our networks by randomly choosing samples from the buffer. Moreover, we predict probabilities for each tile of perimeter twice. In first round we put or remove flags. In second round we use information from uncovered tiles as well as flag information form first round. After predicting probabilities from second round we open the tile with highest probability (of being safe).

Input size is 25 * 12 = 300. We designed three feed-forward neural networks for supervised learning model.  
1. **Small**:  Three hidden layers of sizes 300, 256, and 128, respectively  
2. **Large-A**: Four hidden layers of size 300, 256, 128, and 64, respectively  
3. **Large-B**: Four hidden layers of size 300, 256, 256, and 128, respectively

It is important to point out that this model **scales perfectly** on a board of any size because we always train the same network. That is, training and prediction time is same for all boards.

### Modified Supervised Learning Agent (MSLA)

In some cases it is impossible to say if a given tile contains a mine. Therefore, in the model above we train our network with values that are sometimes **infeasible**. For example, suppose we have two covered tiles with same “true” probability of being safe. Ideally in this case our model should return 0.5 probability for each. However, we always train our model only with 1 or 0. To solve this problem we implemented modified version of supervised learning model. In this modification instead of training with 1 and 0, we use probabilities from other solver. In other words we approximate another solver using neural networks. This approach is useful when computation time of original solver is too long. In this model we used very simple solver based on additive properties of Minesweeper game.

## Results and Analysis

Before testing we trained each of three supervised learning networks on 16x16 board with different number of mines.

<center>
	<img src="/assets/pictures/minesweeper/16x16perf.jpg" alt="Performance on board of size 16 by 16" style="width: 700px;"/>
</center>

We tested all three SLA Neural Networks on 16x16 boards with different mine numbers . From the line chart above we can see that neural networks with 4 hidden layers (Large-A and Large-B) perform better that network with 3 hidden layers (Small) on boards with low density of mines. However, on boards with high density of mines Small performs better than both Large-A and Large-B. In general, **Large-B and Large-A have similar win rates over all mine densities**.

<center>
	<img src="/assets/pictures/minesweeper/8x8perf.jpg" alt="Performance on board of size 8 by 8" style="width: 700px;"/>
</center>

We also tested all three SLA Neural Networks on 8x8 boards with different mine numbers. Small has the highest win rate and Large-A has the lowest win rate over all mine densities. In general, **Large-B perform similar to Large-A**, however, on boards with moderate density it matches with Small. 

These results were quite surprising; we expected that NNs with more layers will perform better since they have higher complexity. However, **more layers take more time to train**. That could be a reason of their poor performance. It is also important how NNs are trained. If NN is trained only on boards with low density of mines, it will have very low win rate on boards with high density of mines.

From the table below we can see that both MSLA and solver for MSLA have low win rate compared to our SLA networks. However, it is worth noticing that MSLA approximates its solver quite well. Therefore, **solver is the bottleneck for MSLA**. This problem could be solved using a better solver to approximate.

<table>
	<thead>
		<tr>
			<th>Board size and # of mines</th>
			<th>Density</th>
			<th>Small</th>
			<th>Large-A</th>
			<th>Large-B</th>
			<th>MSLA</th>
			<th>Solver for MSLA</th>
			<th>Pedersen</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<th>8x8, 8</th>
			<th>12.5%</th>
			<th>69.8%</th>
			<th>65.9%</th>
			<th>66.4%</th>
			<th>30.7%</th>
			<th>33.5%</th>
			<th>92.5%</th>
		</tr>
		<tr>
			<th>8x8, 10</th>
			<th>15.6%</th>
			<th>52.8%</th>
			<th>49.3%</th>
			<th>50.1%</th>
			<th>11%</th>
			<th>15.3%</th>
			<th>-</th>
		</tr>
		<tr>
			<th>10x10, 10</th>
			<th>10%</th>
			<th>-</th>
			<th>-</th>
			<th>-</th>
			<th>39.2%</th>
			<th>40%</th>
			<th>-</th>
		</tr>
		<tr>
			<th>16x16, 20</th>
			<th>7.8%</th>
			<th>83.4%</th>
			<th>87.2%</th>
			<th>87.5%</th>
			<th>40.4%</th>
			<th>47%</th>
			<th>-</th>
		</tr>
		<tr>
			<th>16x16, 40</th>
			<th>15.6%</th>
			<th>40.2%</th>
			<th>33.7%</th>
			<th>34.9%</th>
			<th>0%</th>
			<th>0%</th>
			<th>67.7%</th>
		</tr>
	</tbody>
</table>

Additionally, the table compares our models to limited search with probability estimates strategy ([Pedersen](http://www.minesweeper.info/articles/ComplexityMinesweeperStrategiesForGamePlaying.pdf)) for two boards. In both cases limited search with probability estimates strategy outperforms our models. However, our models, once trained, are faster than the limited search. 
Overall, proposed models show significant improvement compare to previous attempts of using neural networks for MineSweeper agent.

We also implemented convenient interface for the game to visualize the process of ‘thinking’ or estimating probabilities. In our game implementation we used different colors that would represent predicted probabilities. In figure above green color represents “safeness” and purple color represents mines. Using this representation we are able to understand  **how our model “thinks”**.  We can see that in certain tiles our agent is more confident than in others. You can watch [sample video on youtube](https://youtu.be/m93DKCp-P0c).

<center>
	<img src="/assets/pictures/minesweeper/interface.jpg" alt="User Interface" style="width: 500px;"/>
</center>

## Further Research/Work

Since our MSLA approach is based on the approximation of ‘safe’ probability, one obvious way to improve success rate is to use better solver. For instance, we propose to use the **limited search with probability estimates strategy** (K. Pedersen) as a true oracle. Also, this approach will give us a better insight into situations in which it is potentially impossible to be certain about the true probability of the safe choice. This will prevent the cost function from significant fluctuations and possibly improve and speed up learning process. In addition, we believe that it would be beneficial to try to approximate other probabilistic methods, such as SCP, rejection method (for consistent belief state estimation), MCMC, etc.

Find out more by [visiting the project on GitHub](https://github.com/chizhik/minesweeper).