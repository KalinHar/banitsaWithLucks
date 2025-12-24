<template>
	<div class="header">
		<span>Заповядайте на баницата! Моля, не бъдете лакоми, количествата са ограничени!</span>
	</div>
	<div class="banitza">
		<img :class="[spin ? 'banitza-anim' : '', 'banitza-img']" :src="`${lucks.length}.png`"
			@click="inputOpen = true">
	</div>
	<div v-if="inputOpen" class="mask">
		<div class="email-input">
			<div class="close" @click="inputOpen = false">
				<span class="close-email">x</span>
			</div>
			<div class="flex">
				<input type="text" v-model="emailPrefix" placeholder="username" />
				<span type="button" class="email-suffix" @click="choosePiece">{{ emailSuffix }}</span>
			</div>
		</div>
	</div>
	<div v-if="luck" class="mask">
		<div class="luck">
			<div class="close" @click="toCongrats">
				<span>x</span>
			</div>
			<p class="strong">{{ luck }}</p>
		</div>
	</div>

	<div v-if="congrats" class="full">
		<div class="close-grandpa" @click="toCongrats">
			<span>x</span>
		</div>
		<img src="../src/assets/wishes.png" class="congrats">
	</div>

	<div class="lucks">
		<template v-for="email in emails">
			<div v-if="email.luck.length > 0">
				<p><u>{{ email.name }}</u>: {{ email.luck }}</p>
			</div>
		</template>
	</div>
</template>

<script>
const company = import.meta.env.VITE_COMPANY_NAME;
const backendUrl = import.meta.env.VITE_BACKEND_URL;
const aplicationId = import.meta.env.VITE_APP_ID;
const restApiKey = import.meta.env.VITE_REST_API_KEY;

const usersUrl = `${backendUrl}${company}Users`;
const lucksUrl = `${backendUrl}${company}Lucks`;
const btnTextUrl = `${backendUrl}${company}Submit`;
const setLuckUrl = (id) => `${usersUrl}/${id}`;
const headers = {
	'X-Parse-Application-Id': aplicationId,
	'X-Parse-REST-API-Key': restApiKey
}

export default {
	data() {
		return {
			lucks: [],
			luck: '',
			emails: [],
			email: {},
			emailPrefix: "",
			inputOpen: false,
			congrats: false,
			spin: false,
			emailSuffix: '',
			companyLucks: []
		}
	},
	computed: {
		fullEmail() {
			return this.emailPrefix.toLowerCase() + this.emailSuffix;
		}
	},
	async mounted() {
		await this.getCompanyData();
		await this.setLocalState();

		// Clear remote lucks

		// for await (const email of this.emails) {
		//     if (email.luck.length > 0)
		//         fetch(`https://parseapi.back4app.com/classes/vizitec/${email.objectId}`, {
		//             method: 'PUT',
		//             body: JSON.stringify({ luck: '' }),
		//             headers
		//         });
		// }
	},
	methods: {
		async getCompanyData() {
			const lucksResponse = await fetch(lucksUrl, { headers });
			const lucksResult = await lucksResponse.json();
			this.companyLucks = lucksResult.results.map(luck => luck.text);

			const btnTextResponse = await fetch(btnTextUrl, { headers });
			const btnTextResult = await btnTextResponse.json();
			this.emailSuffix = btnTextResult.results[0].btnText;
		},
		async choosePiece() {
			await this.setLocalState();
			const emailIndex = this.emails.findIndex(obj => obj.email === this.fullEmail);

			if (emailIndex > -1) {
				this.inputOpen = false;
				this.email = this.emails[emailIndex]
				this.emails[emailIndex].luck.length
					? alert('Вече си изтеглил късмет от баницата!')
					: this.pickLuck();
			} else {
				alert('Въведен е невалиден e-mail');
			}
		},
		async pickLuck() {
			this.spin = true;
			await this.setLocalState();
			await setTimeout(async () => {
				this.luck = this.lucks[Math.round(Math.random() * (this.lucks.length - 1))];
				await this.setLuckRemote();
				await this.setLocalState();
			}, 2500);
		},
		async setLocalState() {
			this.emails = await this.getAllEmails();
			const usedLucks = this.emails.map(email => email.luck);
			this.lucks = this.companyLucks.filter(luck => !usedLucks.includes(luck));
		},
		async getAllEmails() {
			const emailsResponse = await fetch(usersUrl, { headers });
			const emailsResult = await emailsResponse.json();
			return emailsResult.results;
		},
		async setLuckRemote() {
			const luckResponse = await fetch(setLuckUrl(this.email.objectId), {
				method: 'PUT',
				headers,
				body: JSON.stringify({ luck: this.luck })
			});
		},
		toCongrats() {
			this.luck = '';
			this.congrats = !this.congrats;
			this.spin = false;
		}
	}
}
</script>

<style scoped></style>
