<script setup></script>

<template>
  <div class="instructions-container">
    <h1>App Instructions</h1>

    <h2>Hidden Features:</h2>
    <ul>
      <li>
        By clicking a later episode shine or level unlock all the previous ones also get clicked
      </li>
      <li>
        By clicking shines or blue goals with multiple variants a 2nd time the alternate variant
        gets activated. List of available shines and blue goals
        <a href="https://pastebin.com/q0gZD1DY" target="_blank">here</a>
      </li>
      <li>Unnecessary world rows can be closed by clicking on the world name</li>
      <li>
        If you route in a blue goal over multiple shines faster than it's estimated to take you can
        compensate yourself by adding negative time (say -15) to the misc field
      </li>
      <li>
        You can add shine speeds more fitting for your skill level for more accurate estimates by
        creating your own shine speed file using this
        <a href="https://pastebin.com/pw4kDZZz" target="_blank">template</a> with the file upload
        button.
      </li>
    </ul>

    <h2>Other Notes:</h2>
    <ul>
      <li>
        Blue coin goals assume that you get a shine to accompany it and don't exit area. If you do
        not, add a 30s penalty to misc for the exit to compensate. Blue coin goals don't account for
        routing in blues over multiple episodes. If you save time by doing so feel free to
        compensate yourself by reducing time in misc.
      </li>
      <li>
        Transition movement field does not need movement to level unlocks unless coming from
        unconventionally far away
      </li>
      <li>
        Misc timeloss includes stuff like lives, nozzle unlocks, 50 coins, and blues routed in
        outside specific goals
      </li>
      <li>
        Pianta and Sirena assume there is no pipe animation. If you do them after Pinna, adjust
        accordingly with the misc variable
      </li>
    </ul>

    <router-link to="/" class="back-btn">← Back to route calculator</router-link>
  </div>
</template>

<style scoped>
.instructions-container {
  max-width: 700px;
  margin: 2rem auto;
  padding: 1.5rem 2rem;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
  line-height: 1.6;
  font-size: 1rem;
  color: #333;
}

h1 {
  text-align: center;
  margin-bottom: 1.5rem;
}

h2 {
  margin-top: 1.5rem;
  color: #1e90ff;
}

ul {
  margin: 0.5rem 0 1rem 1.5rem;
  padding: 0;
}

li {
  margin-bottom: 0.6rem;
}

.back-btn {
  display: inline-block;
  margin-top: 1.5rem;
  text-decoration: none;
  background: #1e90ff;
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  transition: background 0.2s;
}

.back-btn:hover {
  background: #1675cc;
}
</style>
