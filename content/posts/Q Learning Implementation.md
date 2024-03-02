---
title: Q Learning Implementation
date: 2024-03-03T03:00:55+08:00
summary: Q Learning Basics and Concepts
tags: [
	"Reinforcement Learning"
]
draft: false
---

### Base Structure
```python
class QLearning:
	self.n_states = 5

	self.action: list of actions
	self.action_size = len(self.action)

	self.alpha = 0.1
	self.gamma = 0.95
	self.epsilon = 1.0
	self.epsilon_min = 0.01
	self.epsilon_decay = 0.995

	self.q_table = pd.DataFrame(np.zeros((self.n_state, self.action_size)),
								columns=self.action)
```

基本上n_state一般來說會是1D, 如我在做trading strategy, 設定的state是前n days daily price changes, 就會是一個 n-d array, 就不適用傳統的Q Learning. QValue 計算的方法就需要用如 function approximator (一個neural network).
### Epsilon Greedy
用來決定exploitation or exploration. 隨著epsilon慢慢的decay, agent會用越來越多的exploitation
```python
def greedy_epsilon(self, state):
	# exploration
    if np.random.rand() <= self.epsilon:
	    return np.argmax(self.q_table[state, :])    
        
    # exploitation
    return random.choice(self.action)
```

### Bellman Equation
Updating Q table base on immediate reward and estimates future reward
<!-- ![asd](/images/123.png) -->
<!-- ![[Pasted image 20240227235622.png]] -->
```python
def learn(self, state, action, reward, next_state):
    q_predict = self.q_table[state, action]
    q_target = reward + self.gamma * np.max(self.q_table[next_state, :])
       
    self.q_table[state, action] += self.alpha * (q_target - q_predict)
```