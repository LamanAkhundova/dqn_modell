import random
import numpy as np
import tensorflow as tf
from collections import deque

class DoubleDQNAgent:
    def __init__(self, state_size, action_size):
        self.state_size = state_size
        self.action_size = action_size
        self.memory = deque(maxlen=2000)
        self.gamma = 0.95  # discount rate
        self.epsilon = 0.1  # exploration rate
        self.epsilon_min = 0.01
        self.epsilon_decay = 0.99
        self.learning_rate = 0.001
        self.model = self._build_model()
        self.target_model = self._build_model()
        self.update_target_model()

    def _build_model(self):
        model = tf.keras.Sequential()
        model.add(tf.keras.layers.Dense(24, input_dim=self.state_size, activation='relu'))
        model.add(tf.keras.layers.Dense(24, activation='relu'))
        model.add(tf.keras.layers.Dense(self.action_size, activation='linear'))
        model.compile(loss='mse', optimizer=tf.keras.optimizers.Adam(lr=self.learning_rate))
        return model

    def update_target_model(self):
        self.target_model.set_weights(self.model.get_weights())

    def remember(self, state, action, reward, next_state, done):
        self.memory.append((state, action, reward, next_state, done))

    def act(self, state):
        if np.random.rand() <= self.epsilon:
            return random.randrange(self.action_size)
        act_values = self.model.predict(state)
        return np.argmax(act_values[0])

    def replay(self, batch_size):
        minibatch = random.sample(self.memory, batch_size)
        for state, action, reward, next_state, done in minibatch:
            target = self.model.predict(state)
            if done:
                target[0][action] = reward
            else:
                a = np.argmax(self.model.predict(next_state)[0])
                t = self.target_model.predict(next_state)[0][a]
                target[0][action] = reward + self.gamma * t
            self.model.fit(state, target, epochs=1, verbose=0)
        if self.epsilon > self.epsilon_min:
            self.epsilon *= self.epsilon_decay

    def load(self, name):
        self.model.load_weights(name)

    def save(self, name):
        self.model.save_weights(name)

# Assuming state_size and action_size are defined
state_size = 4  # Example state size
action_size = 2  # Example action size
agent = DoubleDQNAgent(state_size, action_size)

for episode in range(700):
    state = env.reset()  # Reset environment for new episode
    state = np.reshape(state, [1, state_size])
    for time in range(100):
        action = agent.act(state)  # Select action
        next_state, reward, done, _ = env.step(action)  # Execute action
        reward = reward if not done else -10
        next_state = np.reshape(next_state, [1, state_size])
        agent.remember(state, action, reward, next_state, done)  # Store experience
        state = next_state  # Transition to next state
        if done:
            print(f"Episode: {episode+1}/700, score: {time}, e: {agent.epsilon:.2}")
            break
    if len(agent.memory) > 32:
        agent.replay(32)  # Train the agent with experience replay

# Save the trained model
agent.save("double_dqn_model.h5")
