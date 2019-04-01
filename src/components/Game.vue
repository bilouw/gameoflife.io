<template>
	<div class="container">
		<div style="margin-top: 10px;">
			<table align="center" v-if="worldExist" cellspacing="0">
				<thead>
				</thead>
				<tbody>
					<tr v-for="h in worldSize" :key="h.id">
							<td v-for="w in worldSize" :key="w.id" :style="getCellStyle(h-1, w-1)" @click="setWorldCell(h-1, w-1, myColor)"/>
					</tr>
				</tbody>
			</table>
			<p v-else>Can't retrieve data ...</p>
			</div>
		<div class="row" style="margin-top: 20px;">
			<button class="button-red column column-10 column-offset-25" @click="setMyColor(1)">Red</button>
			<button class="button-green column column-10 column-offset-10" @click="setMyColor(2)">Green</button>
			<button class="button-blue column column-10 column-offset-10" @click="setMyColor(3)">Blue</button>
		</div>
		<div class="row" style="margin-top: 20px;">
			<button class="column column-10 column-offset-25" @click="randomize()">Randomize</button>
			<button class="column column-10 column-offset-10" @click="setPlaying()">Play/Pause</button>
			<button class="column column-10 column-offset-10" @click="cleanWorld()">Clean</button>
		</div>
		<div class="row" style="margin-top: 20px;">
			<input class="column column-10 column-offset-25" type="number" placeholder="World Size" v-model.number="worldSize"  @change="setWorldSize(worldSize)">
			<input class="column column-10 column-offset-10" type="range" min="1" max="20" value="10" v-model.number="cellSize">
			<input class="column column-10 column-offset-10" type="checkbox" v-model="coloredCell">
		</div>
	</div>
</template>

<script>
import Vue from 'vue'

import { Kuzzle, WebSocket } from 'kuzzle-sdk';

export default {
	components: {
		Vue
	},
	data:() => {
		return {
			worldSize: 50,
			cellSize: 10,
			coloredCell: true,
			myColor: 1,
			gen: 0,
			score: 0,
			world: null,
			play: false
		} 
	},
	created: function () {
		this.init();
	},
	mounted: function () {
	},
	computed: {
		worldExist() {
			return this.world == null ? false: true;
		}
	},
	methods: {
		getCellStyle(h, w) {
			let color = '#000';
			if (this.world[h][w]['alive']) {
				switch(this.world[h][w]['color']) {
					case 0: color = '#c4bbaf'; break;
					case 1: color = '#f93943'; break;
					case 2: color = '#248232'; break;
					case 3: color = '#57b8ff'; break;
				}
				if (!this.coloredCell) color = '#fff';
			}
			let size = '; width : ' + this.cellSize + 'px; ' + 'height: ' + this.cellSize + 'px;';
			return 'background-color: ' + color + size;
		},
		setMyColor(color) {
			this.myColor = color;
		},
		update() {
			if(this.play) {
				this.gen++
				this.evolveWorld()
			}
		},
		init() {
			// instantiate a Kuzzle client
			this.$kuzzle = new Kuzzle(new WebSocket('localhost')); // 192.168.0.25 for accessing from outside

			// add a listener to detect any connection problems
			this.$kuzzle.on('networkError', error => {
				console.error(`Network Error: ${error}`);
				this.error = error;
			});
		
			let connect = async () => {
				try {
					// Connect to Kuzzle server
					await this.$kuzzle.connect();
					console.log('Connected to Kuzzle ...');

					//Sub Callback
					const callback = (notification) => {
						this.world = notification.result._source;
					};

					//World Sub
					try {
						await this.$kuzzle.realtime.subscribe('gameoflife', 'worlds', {}, callback, {subscribeToSelf: false});
						console.log('World Subscribed ...');
					} catch (error) {
						console.error(error.message);
					}
				} catch (error) {
					console.error(error.message);
				} finally {
					//this.$kuzzle.disconnect();
				}
			};
			connect();
		},

		async request(query) {
			try {
         const response = await this.$kuzzle.query(query, {});
				 console.log(response._result);
       } catch (error) {
				 console.log(error);
       }
		},

		async evolveWorld() {
			const query = {
         controller: 'kuzzle-core-plugin-gameoflife/worldController',
				 action: 'updateWorld'
			 };

			this.request(query);
		},

		async setWorldCell(y, x, color) {
			const query = {
         controller: 'kuzzle-core-plugin-gameoflife/worldController',
				 action: 'setWorldCell',
				 x: x,
				 y: y,
				 color: color
			 };

			 this.request(query);
		},

		async randomize() {
			const query = {
         controller: 'kuzzle-core-plugin-gameoflife/worldController',
         action: 'randomizeWorld'
			 };
			 
			 this.request(query);
		},

		async setPlaying() {
			const query = {
         controller: 'kuzzle-core-plugin-gameoflife/worldController',
         action: 'setPlaying'
			 };
			 
			 this.request(query);
		},

		async cleanWorld() {
			const query = {
         controller: 'kuzzle-core-plugin-gameoflife/worldController',
         action: 'cleanWorld'
			 };
			 
			 this.request(query);
		},

		async setWorldSize(size) {
			const query = {
         controller: 'kuzzle-core-plugin-gameoflife/worldController',
				 action: 'setWorldSize',
				 size: size
			 };

			 this.request(query);
		}
	}
}
</script>

<style src="./milligram.css"/>
