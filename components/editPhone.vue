<template>
	<div class="mb-6 text-center">
		<div v-if="edit">
			<select
				id="countryCodes"
				v-model="codeRaw"
				:disabled="enviado"
				name="codes"
				class="my-4 appearance-none px-4 py-2 text-center border-2 rounded-2xl border-yellow-400 w-24">
				<option :value="codigoVE.code" selected>
					<p>{{getCode(codigoVE)}}</p>
				</option>
				<option
					v-for="(country, index) in countriesCodes"
					:key="index"
					:value="country.code">
					<p>{{getCode(country)}}</p>
				</option>
			</select>
			<input
				v-model="numberRaw"
				:disabled="enviado"
				type="string"
				class="px-4 py-2 text-center border-2 rounded-2xl border-yellow-400"/><br />
			<button
				v-if="edit"
				:disabled="enviado"
				class="text-sm bg-[#501111] rounded-3xl px-4 py-2 text-white hover:text-yellow-400"
				@click="makeid()">
				ENVIAR CÓDIGO
			</button>
			<p v-if="enviado" class="text-sm mt-2">
				Código enviado, espere unos minutos antes de intentar de nuevo. Para
				intentar de nuevo haga click en Cancelar
			</p>
		</div>
		<h3 v-if="!edit" class="px-4 py-2 text-center text-xl">
			{{ user.telefono }}
		</h3>

		<h4 v-if="edit" class="mb-2 mt-6">
			Ingrese el código secreto enviado a este numero de WhatsApp
		</h4>
		<input
			v-if="edit"
			v-model="secretConfirm"
			type="string"
			class="px-4 py-2 text-center border-2 rounded-2xl border-yellow-400"/>

		<div class="mt-4">
			<button
				v-if="!edit"
				class="text-sm bg-[#501111] rounded-3xl px-4 py-2 text-white hover:text-yellow-400 mt-2"
				@click="edit = true">
				EDITAR WHATSAPP
			</button>
			<button
				v-if="edit"
				class="text-sm bg-[#501111] rounded-3xl px-4 py-2 text-white hover:text-yellow-400 mt-2"
				@click="editarInfoTelefono">
				GUARDAR
			</button>
			<button
				v-if="edit"
				class="text-sm bg-[#501111] rounded-3xl px-4 py-2 text-white hover:text-yellow-400"
				@click="cancel()">
				CANCELAR
			</button>
		</div>
	</div>
</template>

<script>
import { countries } from '../static/countryCode.json'
export default {
	props: {
		open: {
			type: Boolean,
			required: false,
			default: false,
		},
	},
	data() {
		return {
			edit: this.open,
			countriesCodes: countries,
			enviado: false,
			codeRaw: '+58',
			numberRaw: '',
			secret: '',
			secretConfirm: null,
			codigoVE: {
				code: "+58",
				name: "Venezuela"
			},
		}
	},
	computed: {
		user() {
			return { ...this.$store.state.user }
		},
	},
	methods: {
		getCode(country){
			if(this.codeRaw === country.code){
				return country.code
			} 
			return `${country.name} ${country.code}`	
		},
		makeid() {
			if (this.numberRaw.length !== 10) {
				alert(
					'El número debe tener 10 dígitos sin espacios\nRecuerde colocar el código sin 0\nEjemplo: 4141234567'
				)
				return
			}
			if (this.codeRaw.length === 0) {
				alert('Seleccione el código de país')
				return
			}
			let result = ''
			const number = `${this.codeRaw}${this.numberRaw}`
			const characters =
				'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
			const charactersLength = characters.length
			for (let i = 0; i < 6; i++) {
				result += characters.charAt(Math.floor(Math.random() * charactersLength))
			}
			this.secret = result
			const db = this.$fireModule.firestore()
			db.collection('messages').add({
				to: `whatsapp:${number}`,
				body: `Bienvenido a NATUPETS, su código de confirmación es: ${result}`,
			})
			this.enviado = true
		},
		editarInfoTelefono() {
			if (this.secretConfirm !== this.secret) {
				alert('El código secreto no coincide')
				return
			}
			this.user.telefono = `${this.codeRaw}${this.numberRaw}`
			const db = this.$fireModule.firestore()
			db.collection('users').doc(this.user.id).update({
				telefono: this.user.telefono,
				// cambiar esto por una lista de verificados
				verificado: true,
			})
			this.$store.commit('setUser', this.user)
			this.edit = false
			this.enviado = false
		},
		cancel() {
			this.edit = false
			this.enviado = false
		},
	},
}
</script>

<style></style>
