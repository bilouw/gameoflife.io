<template>
	<div class="container">
		<div class="row">
			<h1 class="column column-offset-25">Game Of Life Online</h1>
		</div>
		<div class="row" style="margin-top: 10px;">
			<table class="column column-50 column-offset-25" v-if="worldExist" cellspacing="0">
				<thead>
				</thead>
				<tbody>
					<tr v-for="h in height" :key="h.id">
							<td v-for="w in width" :key="w.id" :style="getCellColor(h-1, w-1)" class="cell" @click="setWorldCell(h-1, w-1)"/>
					</tr>
				</tbody>
			</table>
			<p v-else>Can't retrieve data ...</p>
			</div>
		<div class="row" style="margin-top: 20px;">
			<button class="column column-10 column-offset-25" @click="randomize()">Randomize</button>
		</div>
	</div>
</template>

<script>
import Vue from 'vue'
import ControlPanel from './ControlPanel'

import { Kuzzle, WebSocket } from 'kuzzle-sdk';

export default {
	components: {
		Vue,
		ControlPanel
	},
	data:() => {
		return {
			width: 50,
			height: 50,
			gen: 0,
			score: 0,
			world: null,
			play: false
		} 
	},
	created: function () {
		this.init();
		this.$timer = setInterval(() => {
			this.update()
		}, 200);
	},
	mounted: function () {
	},
	computed: {
		worldExist() {
			let ret = true;
			if (this.world == null) ret = false;
			return ret;
		}
	},
	methods: {
		getCellColor(h, w) {
			let color = '#000';
			//console.log(this.world[h][w])
			if (this.world[h][w]['alive']) {
				switch(this.world[h][w]['color']) {
					case 0: color = '#C4BBAF'; break;
					case 1: color = '#F93943'; break;
					case 2: color = '#248232'; break;
					case 3: color = '#57B8FF'; break;
				}
			}
			return 'background-color: ' + color;
		},
		update() {
			if(this.play) {
				this.gen++
				this.evolveWorld()
			}
		},
		init() {
			// instantiate a Kuzzle client
			this.$kuzzle = new Kuzzle(new WebSocket('163.172.174.150')); // 192.168.0.25 for accessing from outside

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

					//Get the world
					this.getWorld();

					//Sub Callback
					const callback = (error, notification) => {
						this.getWorld();
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
		async getWorld() {
			const response = await this.$kuzzle.document.get('gameoflife', 'worlds', 'world1');
			this.world = response._source;
		},

		async evolveWorld() {
			const query = {
         controller: 'kuzzle-core-plugin-gameoflife/worldController',
				 action: 'updateWorld'
			 };

			try {
         const response = await this.$kuzzle.query(query, {});
				 console.log(response._result);
       } catch (error) {
				 console.log(error);
       }
		},

		async setWorldCell(y, x) {
			const query = {
         controller: 'kuzzle-core-plugin-gameoflife/worldController',
				 action: 'setWorldCell',
				 x: x,
				 y: y
			 };

			 try {
         const response = await this.$kuzzle.query(query, {});
				 console.log(response._result);
       } catch (error) {
				 console.log(error);
       }

		},

		async randomize() {
			const query = {
         controller: 'kuzzle-core-plugin-gameoflife/worldController',
         action: 'randomizeWorld'
			 };
			 
			 try {
         const response = await this.$kuzzle.query(query, {});
				 console.log(response._result);
       } catch (error) {
				 console.log(error);
       }
		},
		destroyed: function () {
			clearInterval(this.$timer)
		}	
	}
}
</script>

<style src="./milligram.css"/>
