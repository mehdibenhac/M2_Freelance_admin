<template>
	<div>
		<div class="ui basic segment">
			<form class="ui form">
				<div class="five fields">
					<div class="field">
						<div class="ui icon left labeled input">
							<div class="ui blue label">
								<i class="hashtag icon"></i>
							</div>
							<input type="text" v-model="offreFilter.ID" placeholder="ID...">
							<i class="search icon"></i>
						</div>
					</div>
					<div class="field">
						<div class="ui icon labeled input">
							<div class="ui blue label">
								<i class="write icon"></i>
							</div>
							<input type="text" v-model="offreFilter.titre" placeholder="Titre...">
							<i class="search icon"></i>
						</div>
					</div>
					<div class="field">
						<div class="ui icon labeled input">
							<div class="ui blue label">
								<i class="student icon"></i>
							</div>
							<input type="text" v-model="offreFilter.competence" placeholder="Compétence...">
							<i class="search icon"></i>
						</div>
					</div>
					<div class="field">
						<div class="ui icon labeled input">
							<div class="ui blue label">
								<i class="user icon"></i>
							</div>
							<input type="text" v-model="offreFilter.employeur" placeholder="Employeur...">
							<i class="search icon"></i>
						</div>
					</div>
					<div class="field">
						<div class="ui icon labeled input">
							<div class="ui blue label">
								<i class="maps pin icon"></i>
							</div>
							<input type="text" v-model="offreFilter.localisation" placeholder="Localisation...">
							<i class="search icon"></i>
						</div>
					</div>
				</div>
				<div class="ui center aligned container">
					<div class="inline field">
						<label> Etat de l'offre: </label>
						<div class="ui toggle checkbox">
							<input type="radio" v-model="offreFilter.etat" value="Ouverte">
							<label><i class="green circle outline icon"></i>Ouverte</label>
						</div>
						&nbsp;&nbsp;&nbsp;&nbsp;
						<div class="ui toggle checkbox">
							<input type="radio" v-model="offreFilter.etat" value="Fermée">
							<label><i class="red lock icon"></i>Fermée</label>
						</div>
						&nbsp;&nbsp;&nbsp;&nbsp;
						<div class="ui toggle checkbox">
							<input type="radio" v-model="offreFilter.etat" value="Négociation">
							<label><i class="orange wait icon"></i>En négociation</label>
						</div>
					</div>
				</div>
				<div class="ui divider"></div>
				<div class="ui text container"> <b>{{filteredOffres.length}}</b> <i class="blue large newspaper icon"></i>Offre(s) trouvée(s).</div>
			</form>
			<div class="ui inverted divider">
			</div>
		</div>
		<transition :css="false" @enter="flashAnim" @leave="flashAnim" mode="out-in">
			<div class="ui success icon message" v-if="modifiedOffre" key="01">
				<i class="close icon" @click="closeMessage"></i>
				<i class="checkmark icon"></i>
				<div class="content">
					<div class="header"> Offre modifiée</div>
					L'offre <b>{{modifiedOffre._id}}</b> a été modifiée.
				</div>
			</div>
			<div class="ui success icon message" v-if="deletedOffre" key="02">
				<i class="close icon" @click="closeMessage"></i>
				<i class="checkmark icon"></i>
				<div class="content">
					<div class="header">
						Offre supprimée
					</div>
					L'offre <b>{{deletedOffre.offre._id}}</b> a été supprimée. <b>{{deletedOffre.contrat.n}}</b> contrat supprimé.
				</div>
			</div>
		</transition>
		<transition :css="false" @enter="slideDownAnim" @leave="slideUpAnim" mode="out-in">
			<div class="ui info icon message" v-if="filteredOffres.length === 0" key="visible">
				<i class="info icon"></i>
				<div class="header">
					Aucune offre à afficher.
				</div>
			</div>
			<div v-else key="invisible">
				<transition-group tag="div" :css="false" @enter="slideDownAnim" @leave="slideUpAnim">
					<Offre v-for="(offre, index) in filteredOffres" v-if="offre._id !== notToBlur && blur" :showCard="false" :offre="offre" :index="index +1"
					    :key="offre._id" @modAppear="blurOthers" />
					<Offre v-else :showCard="true" :offre="offre" :index="index +1" :key="offre._id" @modAppear="blurOthers" @modified="handleModified"
					    @deleted="handleDeleted" />
				</transition-group>
			</div>
		</transition>
	</div>
</template>

<script>
	import Offre from './Offre'
	import {
		slideUpAnim,
		slideDownAnim,
		flashAnim
	} from '../../../transitions.js'
	import {
		bus
	} from '../../../main'

	export default {
		props: ['offres'],
		data() {
			return {
				notToBlur: '',
				blur: false,
				modifiedOffre: null,
				deletedOffre: null,
				offreFilter: {
					ID: '',
					employeur: '',
					competence: '',
					titre: '',
					localisation: '',
					etat: 'Ouverte',
				}
			}
		},
		computed: {
			filteredOffres: function () {
				let idPat = new RegExp(this.offreFilter.ID.toLowerCase());
				let titrePat = new RegExp(this.offreFilter.titre.toLowerCase());
				let compPat = new RegExp(this.offreFilter.competence.toLowerCase());
				let empPat = new RegExp(this.offreFilter.employeur.toLowerCase());
				let localPat = new RegExp(this.offreFilter.localisation.toLowerCase());
				return this.offres.filter((offre) => {
					return (idPat.test(offre._id.toLowerCase()) && titrePat.test(offre.titre.toLowerCase()) && empPat.test(offre.employeur
							.nom.toLowerCase()) &&
						compPat.test(offre.competence.titre.toLowerCase()) && localPat.test(offre.localisation.toLowerCase()) &&
						offre.etat === this.offreFilter.etat)
				})
			}
		},
		methods: {
			slideUpAnim: function (el, done) {
				slideUpAnim(el, done)
			},
			slideDownAnim: function (el, done) {
				slideDownAnim(el, done)
			},
			flashAnim: function (el, done) {
				flashAnim(el, done)
			},
			blurOthers: function (id) {
				if (this.notToBlur !== "") {
					this.notToBlur = ""
					this.blur = false;
				} else {
					this.notToBlur = id;
					this.blur = true;
				}
			},
			handleModified: function (data) {
				this.deletedOffre = null;
				this.modifiedOffre = data;
				this.offreFilter.ID = '';
				this.offreFilter.titre = '';
				this.offreFilter.employeur = '';
				this.offreFilter.competence = '';
				this.$emit('refresh');
			},
			handleDeleted: function (data) {
				this.modifiedOffre = null;
				this.deletedOffre = data;
				console.log(data);
				this.offreFilter.ID = '';
				this.offreFilter.titre = '';
				this.offreFilter.employeur = '';
				this.offreFilter.competence = '';
				this.$emit('refresh');
			},
			closeMessage: function (el, done) {
				this.modifiedOffre = null;
				this.deletedOffre = null;
			}
		},
		watch: {
			filteredOffres: function () {
				this.blur = false;
				this.notToBlur = "";
				bus.$emit('filterChanged');
			}
		},
		components: {
			Offre
		}
	}

</script>
<style scoped>
	.ui.basic.segment {
		padding: 0;
	}

	.ui.basic.disabled.segment {
		padding: 0;
	}

	.ui.blue.label i {
		margin: 0;
	}

</style>
