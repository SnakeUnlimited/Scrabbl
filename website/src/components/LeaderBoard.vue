
<template>
    <div class="leaderboard">
        <h2>Leaderboard</h2>
        <ul>
            <li v-for="(player, index) in players.stats" :key="index">
                {{ player.name }}: {{ player.score }} points
            </li>
        </ul>
    </div>
</template>

<script>






export default {
    name: 'HelloWorld',
    props: {
        players: Object
    }
}
</script>






<style>
.leaderboard {
    background-color: #f0f0f0;
    padding: 20px;
    border-radius: 10px;
    width: 200px;
    text-align: center;
    color: #333;
    margin-right: 20px;}
</style>