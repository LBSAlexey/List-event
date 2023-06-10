<script>
import axios from "axios";
axios.defaults.baseURL = 'http://localhost:3005';
import dayjs from 'dayjs';
import 'dayjs/locale/ru';

// Подключаем библиотеку 
// и перевод дат на русский язык
dayjs.locale('ru');




export default {


	data() {
		return {
			// события
			events: [],

			month: 'all',

			selectEvent: [],

			show: false,

		};
	},

	methods: {
		async loadEvents() {
			let responce = await axios.get("/events/all");
			this.events = responce.data
		},

		getDate(isoDate) {
        // создание даты из ISO строки
        let date = dayjs(isoDate);

        // перевод в читаемый вид
		let format = date.format("DD MMMM");

		return format;
		},


		getTime(isoDate) {
        // создание даты из ISO строки
        let date = dayjs(isoDate);

        // перевод в читаемый вид
		let format = date.format("HH:mm");

		return format;
		},


		search() {
			if (this.month == 'all') {
				this.loadEvents();
			} else if (this.month == 'next') {
				this.loadNext();
			} else if (this.month == 'june') {
				this.loadJune();
			} else if(this.month == 'july') {
				this.loadJuly();
			}
		},

		async loadNext() {
			let from = dayjs().toISOString();

			let response = await axios.get("/events/next", { params: {
				from: from
			}});

			this.events = response.data;
		},

		async loadJune() {
			let from = dayjs("2023-06-01").toISOString();
			let to = dayjs("2023-06-31").toISOString();

			let response = await axios.get("/events/next", { params: {
				from: from,
				to: to
			}});

			this.events = response.data;
		},

		async loadJuly() {
			let from = dayjs("2023-07-01").toISOString();
			let to = dayjs("2023-07-31").toISOString();

			let response = await axios.get("/events/next", { params: {
				from: from,
				to: to
			}});

			this.events = response.data;
		},


		async selectLoad(item, index) {
			this.selectEvent = this.events[index];
			this.show = true;

			let responce = await axios.get(`/event/show`, {params: {
				id: item._id
			}})			

			this.selectEvent = responce.data;
			
		},

		async register(event, index) {
			let responce = await axios.get(`/events/register`, {params: {
				id: event._id
			}})			


			this.selectEvent = responce.data;

			return false;
		}
	},

	mounted() {
		this.loadEvents();
	}
};
</script>

<template>
	<div class="app">
		<div class="container my-3">
			<div class="row row-cols-2 justify-content-center">

				<!-- Список событий -->
				<div class="col">
					<!-- Поле поиска -->
					<div class="input-group mb-3">
						<select class="form-select form-select-lg" v-model="month" @change="search">
							<option value="all">Все события</option>
							<option value="next">Скоро</option>
							<option value="june">Июнь</option>
							<option value="july">Июль</option>
						</select>
					</div>

					<!-- Карточки событий -->
					<ul class="list-group">
						<li class="list-group-item" :class="{selected: selectEvent == item}" v-for="(item, index) in events" @click="selectLoad(item, index)">
							<div class="event-date">
								{{ getDate(item.date) }}
							</div>

							<div class="event-title">
								{{ item.title }}
							</div>
						</li>
					</ul>
				</div>
				<div class="col show-event" v-show="show">
					<div class="card mb-3">
						<div class="card-body">
							<h2 class="card-title">
								{{ selectEvent.title }}
							</h2>
							<p class="card-text">
								{{ selectEvent.description }}.
							</p>
							<p class="card-text">
							<div class="event-time">
								{{ getDate(selectEvent.date) }} в {{ getTime(selectEvent.date) }}
							</div>
				
							<div class="event-tickets">
								<div>
									<p>
										Свободных мест: {{ selectEvent.ticketsCount }}
									</p>
				
									<button class="btn btn-outline-primary" v-show="!selectEvent.registed"  @click="register(selectEvent)">
										Зарегистрироваться
									</button>
								</div>
							</div>
				
				
							</p>
						</div>
						<img :src="'src/assets/' + selectEvent.image" class="card-img-bottom" />
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
body {
	background-color: lightgray;
}

.search-input {
	font-size: 40px;
	text-align: center;
}

.list-group-item {
	display: flex;
	cursor: pointer;
	margin: 4px 0;
	border: none;
	border-radius: 4px;
	/* background-color: aliceblue; */
	/* border-top: 1px; */
}

.list-group-item.selected {
	outline: 2px solid blueviolet;
}

.list-group-item .event-date {
	width: 100px;
	font-size: 30px;
	/* color: white; */
	/* background-color: #0d6efd; */
	border-radius: 4px;

	display: flex;
	justify-content: center;
	align-items: center;
}

.list-group-item .event-title {
	flex: 1;
	padding: 0 14px;
}

.show-event .event-time {
	font-size: 30px;
	margin-bottom: 10px;
}

</style>
