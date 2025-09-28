'use strict';

const auction = {
  // Config: starting money, min players, etc
  startingMoney: 100,
  minPlayers: 2,
  bids: {},
  players: [],
  auctionActive: false,
  highestBid: 0,
  highestBidder: null,

  init(players) {
    this.players = players;
    this.bids = {};
    this.auctionActive = true;
    this.highestBid = 0;
    this.highestBidder = null;

    for (const p of players) {
      this.bids[p] = 0;
    }
  },

  placeBid(player, amount, room) {
    if (!this.auctionActive) return room.send("No auction is running.");
    if (!this.players.includes(player)) return room.send(`${player}, you are not part of the auction.`);
    if (amount <= this.highestBid) return room.send(`${player}, your bid must be higher than the current highest bid (${this.highestBid}).`);

    this.bids[player] = amount;
    this.highestBid = amount;
    this.highestBidder = player;
    room.send(`${player} bids ${amount} points! Highest bid is now ${amount} by ${player}.`);
  },

  endAuction(room) {
    if (!this.auctionActive) return room.send("No auction to end.");
    this.auctionActive = false;
    if (this.highestBidder) {
      room.send(`Auction ended! Winner is ${this.highestBidder} with a bid of ${this.highestBid} points.`);
    } else {
      room.send("Auction ended with no bids placed.");
    }
  }
};

exports.commands = {
  auction: {
    init(target, room, user) {
      if (!this.can('mute', null, room)) return false; // only mods+
      const players = target.split(',').map(p => p.trim());
      if (players.length < auction.minPlayers) return this.errorReply(`Need at least ${auction.minPlayers} players.`);
      auction.init(players);
      room.send(`Auction started with players: ${players.join(', ')}. Starting money: ${auction.startingMoney}`);
    },

    bid(target, room, user) {
      if (!auction.auctionActive) return this.errorReply("No auction is running.");
      const amount = parseInt(target);
      if (isNaN(amount) || amount <= 0) return this.errorReply("Please enter a valid bid amount.");
      auction.placeBid(user.name, amount, room);
    },

    end(target, room, user) {
      if (!this.can('mute', null, room)) return false;
      auction.endAuction(room);
    }
  }
};# Auction-
Poketails auction bot
