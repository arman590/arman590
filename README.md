 const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server);

app.use(express.static('public')); // د مخکیني فایلونو لپاره د عامه فولډر وکاروئ

const players = {}; // د لوبغاړو ساتلو لپاره

io.on('connection', (socket) => {
  console.log(`User connected: ${socket.id}`);
  
  // نوی لوبغاړی اضافه کول
  socket.on('new-player', (playerData) => {
    players[socket.id] = playerData;
    socket.broadcast.emit('player-joined', { id: socket.id, ...playerData });
  });

  // د لوبغاړي موقعیت تازه کول
  socket.on('player-move', (data) => {
    if (players[socket.id]) {
      players[socket.id].x = data.x;
      players[socket.id].y = data.y;
      socket.broadcast.emit('player-moved', { id: socket.id, x: data.x, y: data.y });
    }
  });

  // د لوبغاړي پریښودل
  socket.on('disconnect', () => {
    console.log(`User disconnected: ${socket.id}`);
    delete players[socket.id];
    socket.broadcast.emit('player-left', { id: socket.id });
  });
});

server.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
