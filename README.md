const express = require("express");
const app = express();
const http = require("http").createServer(app);
const io = require("socket.io")(http);

app.use(express.static("public"));

let players = {};

io.on("connection", (socket) => {
    console.log("Player connected:", socket.id);

    players[socket.id] = {
        x: 100,
        y: 100
    };

    socket.on("move", (data) => {
        if (players[socket.id]) {
            players[socket.id].x = data.x;
            players[socket.id].y = data.y;
        }
    });

    socket.on("disconnect", () => {
        console.log("Player disconnected:", socket.id);
        delete players[socket.id];
    });
});

setInterval(() => {
    io.emit("players", players);
}, 1000 / 60);

http.listen(3000, () => {
    console.log("Server running on port 3000");
});
<!DOCTYPE html>
<html>
<head>
    <title>Snowball Shooter</title>
    <style>
        canvas {
            background: black;
        }
    </style>
</head>
<body>
    <canvas id="game" width="800" height="600"></canvas>

    <script src="/socket.io/socket.io.js"></script>
    <script>
        const socket = io();
        const canvas = document.getElementById("game");
        const ctx = canvas.getContext("2d");

        let players = {};
        let myPlayer = { x: 100, y: 100 };

        document.addEventListener("mousemove", (e) => {
            myPlayer.x = e.clientX;
            myPlayer.y = e.clientY;

            socket.emit("move", myPlayer);
        });

        socket.on("players", (serverPlayers) => {
            players = serverPlayers;
        });

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            for (let id in players) {
                const p = players[id];

                ctx.fillStyle = "white";
                ctx.fillRect(p.x, p.y, 20, 20);
            }

            requestAnimationFrame(draw);
        }

        draw();
    </script>
</body>
</html>
npm init -y
npm install express socket.io
node server.js
