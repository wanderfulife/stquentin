<template>
	<div class="container">

		<div id="enqueteur" v-if="level === 0">
			<h2>Prénom enqueteur :</h2>
			<input class="form-control" type="text" v-model="ENQUETEUR" />
			<button v-if="ENQUETEUR" @click="next" class="btn-next">Suivant</button>
		</div>

		<div id="start" v-if="level === 1">
			<button @click="startSurvey" class="btn-next">COMMENCER QUESTIONNAIRE</button>
		</div>

		<div id="q1" v-if="level === 2">
			<h1>Nombre de personnes dans le véhicule</h1>
			<input class="form-control" type="text" v-model="Q1" placeholder="Precisions">
			<button v-if="Q1" @click="next" class="btn-next">Suivant</button>
			<button @click="back" class="btn-return">retour</button>
		</div>

		<div id="q2" v-if="level === 3">
			<h1> Pour qelle(s) raison(s) êtes vous garés ici ? </h1>
			<select v-model="Q2" class="form-control">
				<option v-for="option in motifstationnement" :key="option.id" :value="option.output">
					{{ option.text }}
				</option>
			</select>
			<input v-if="Q2 === 8" class="form-control" type="text" v-model="Q2_DETAIL" placeholder="Precisions">
			<button v-if="Q2" @click="next" class="btn-next">Suivant</button>
			<button @click="back" class="btn-return">retour</button>
		</div>

		<div id="q3" v-if="level === 4">
			<h1>Combien de temps prévoyez-vous de rester garé ici ? </h1>
			<select v-model="Q3" class="form-control">
				<option v-for="option in dureestationnement" :key="option.id" :value="option.output">
					{{ option.text }}
				</option>
			</select>
			<input v-if="Q3 === 6" class="form-control" type="text" v-model="Q3_DETAIL" placeholder="Precisions">
			<button v-if="Q3" @click="next" class="btn-next">Suivant</button>
			<button @click="back" class="btn-return">retour</button>
		</div>

		<div id="q4" v-if="level === 5">
			<h1>Vous stationnez ici ? </h1>
			<select v-model="Q4" class="form-control">
				<option v-for="option in frequencestationnement" :key="option.id" :value="option.output">
					{{ option.text }}
				</option>
			</select>
			<input v-if="Q4 === 8" class="form-control" type="text" v-model="Q4_DETAIL" placeholder="Precisions">
			<button v-if="Q4" @click="next" class="btn-next">Suivant</button>
			<button @click="back" class="btn-return">retour</button>
		</div>

		<div id="q5" v-if="level === 6">
			<h1>Quelle est votre commune de résidence </h1>
			<div>
				<CommuneSelector v-model="Q5" />
			</div>
			<input v-if="Q5 === 'MONTIGNY LE BRETONNEUX - 78423'" class="form-control" type="text" v-model="Q5_DETAIL"
				placeholder="Dans quel quartier ?">
			<button v-if="Q5" @click="next" class="btn-next">Suivant</button>
			<button @click="back" class="btn-return">retour</button>
		</div>

		<div id="end" v-if="level === 7">
			<button @click="submitSurvey" class="btn-next">FINIR QUESTIONNAIRE</button>
			<button @click="back" class="btn-return">retour</button>
		</div>
		<img class="logo" src="../assets/Alycelogo.webp" alt="Logo Alyce">

		<button class="btn-fin" @click="downloadData">download DATA</button>

	</div>
	<div>
		<span style="margin-left: 10px;">Nombre de questionnaires : {{ docCount }}</span>
	</div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import {
	motifstationnement,
	dureestationnement,
	frequencestationnement,
} from "./reponses";
import GareSelector from "./GareSelector.vue";
import CommuneSelector from './CommuneSelector.vue';
import { db } from "../firebaseConfig";
import { collection, getDocs, addDoc } from "firebase/firestore";
import * as XLSX from "xlsx";

const docCount = ref(0);
const surveyCollectionRef = collection(db, "StQuentin");
const level = ref(0);
const startDate = ref('');
const ENQUETEUR = ref('');
const Q1 = ref('');
const Q2 = ref('');
const Q2_DETAIL = ref('');
const Q3 = ref('');
const Q3_DETAIL = ref('');
const Q4 = ref('');
const Q4_DETAIL = ref('');
const Q5 = ref('');
const Q5_DETAIL = ref('');



const startSurvey = () => {
	startDate.value = new Date().toLocaleTimeString("fr-FR").slice(0, 8);
	level.value++;
}


const next = () => {
	level.value++;
	console.log(level.value)
}

const back = () => {
	level.value--;
}


const getDocCount = async () => {
	try {
		const querySnapshot = await getDocs(surveyCollectionRef);
		docCount.value = querySnapshot.size;
	} catch (error) {
		console.error("Error getting document count:", error);
	}
};

onMounted(getDocCount);

const submitSurvey = async () => {
	await addDoc(surveyCollectionRef, {
		HEURE_DEBUT: startDate.value,
		DATE: new Date().toLocaleDateString("fr-FR").replace(/\//g, "-"),
		JOUR: new Date().toLocaleDateString("fr-FR", { weekday: 'long' }),
		ENQUETEUR: ENQUETEUR.value,
		HEURE_FIN: new Date().toLocaleTimeString("fr-FR").slice(0, 8),
		Q1: Q1.value,
		Q2: Q2.value,
		Q2_DETAIL: Q2_DETAIL.value,
		Q3: Q3.value,
		Q3_DETAIL: Q3_DETAIL.value,
		Q4: Q4.value,
		Q4_DETAIL: Q4_DETAIL.value,
		Q5: Q5.value,
		Q5_DETAIL: Q5_DETAIL.value,
	});
	level.value = 1;
	startDate.value = "";
	Q1.value = "";
	Q2.value = "";
	Q2_DETAIL.value = "";
	Q3.value = "";
	Q3_DETAIL.value = "";
	Q4.value = "";
	Q4_DETAIL.value = "";
	Q5.value = "";
	Q5_DETAIL.value = "";
	getDocCount();
};

const downloadData = async () => {
	try {
		const querySnapshot = await getDocs(surveyCollectionRef);
		let data = [];
		let maxWidths = {}; // Object to keep track of maximum width for each column

		// Define your headers // MODIFICATION SUR L'EXCEL
		const headers = {
			ID_questionnaire: "ID_questionnaire",
			ENQUETEUR: "ENQUETEUR",
			DATE: "DATE",
			JOUR: "JOUR",
			HEURE_DEBUT: "HEURE_DEBUT",
			HEURE_FIN: "HEURE_FIN",
			Q1: "Q1",
			Q2: "Q2",
			Q2_DETAIL: "Q2_DETAIL",
			Q3: "Q3",
			Q3_DETAIL: "Q3_DETAIL",
			Q4: "Q4",
			Q4_DETAIL: "Q4_DETAIL",
			Q5: "Q5",
			Q5_DETAIL: "Q5_DETAIL",
		};

		// Initialize maxWidths with header lengths
		Object.keys(headers).forEach((key) => {
			maxWidths[key] = headers[key].length;
		});

		querySnapshot.forEach((doc) => {
			let docData = doc.data();
			let mappedData = {
				ID_questionnaire: doc.id,
				ENQUETEUR: docData.ENQUETEUR || "",
				DATE: docData.DATE || "",
				JOUR: docData.JOUR || "",
				HEURE_DEBUT: docData.HEURE_DEBUT || "",
				HEURE_FIN: docData.HEURE_FIN || "",
				Q1: docData.Q1 || "",
				Q2: docData.Q2 || "",
				Q2_DETAIL: docData.Q2_DETAIL || "",
				Q3: docData.Q3 || "",
				Q3_DETAIL: docData.Q3_DETAIL || "",
				Q4: docData.Q4 || "",
				Q4_DETAIL: docData.Q4_DETAIL || "",
				Q5: docData.Q5 || "",
				Q5_DETAIL: docData.Q5_DETAIL || "",
			};
			data.push(mappedData);
			// Update maxWidths for each key in mappedData
			Object.keys(mappedData).forEach((key) => {
				const valueLength = mappedData[key].toString().length;
				maxWidths[key] = Math.max(maxWidths[key], valueLength);
			});
		});
		// Convert data to a worksheet
		const worksheet = XLSX.utils.json_to_sheet(data, {
			header: Object.keys(headers),
			skipHeader: false,
		});
		// Set the widths for each column
		worksheet["!cols"] = Object.keys(maxWidths).map((key) => ({
			wch: maxWidths[key] + 2 // +2 for a little extra padding
		}));
		const workbook = XLSX.utils.book_new();
		XLSX.utils.book_append_sheet(workbook, worksheet, "Data"); ``
		// Export the workbook to a .xlsx file
		XLSX.writeFile(workbook, "StQuentin.xlsx");
	} catch (error) {
		console.error("Error downloading data: ", error);
	}
};

</script>

<style>
body {
	background-color: #2a3b63;
}

.logo {
	padding: 10%;
	height: 3em;
}

h1 {
	text-align: center;
	color: #4caf50;
	font-size: 18px;
}

h2 {
	color: white;
	font-size: 16px;
}

.english {
	color: cyan;
}

.container {
	background-color: #2a3b63;
	color: white;
	padding: 5% 0;
	width: 75%;
	margin: auto;
}

.btn-next {
	width: 100%;
	background-color: green;
	color: white;
	padding: 20px 20px;
	margin-top: 20%;
	border: none;
	border-radius: 5px;
	cursor: pointer;
}


.btn-fin {
	width: 100%;
	background-color: #4c4faf;
	color: white;
	padding: 20px 20px;
	margin-top: 5%;
	border: none;
	border-radius: 5px;
	cursor: pointer;
}

.btn-return {
	width: 100%;
	background-color: #898989;
	color: white;
	padding: 20px 20px;
	margin-top: 5%;
	border: none;
	border-radius: 5px;
	cursor: pointer;
}

.btn-return:hover {
	background-color: #839684;
}

.commune-dropdown {
	/* Style your dropdown list here */
	list-style-type: none;
	padding: 0;
	margin: 0;
	border: 1px solid #ccc;
	border-radius: 4px;
	max-height: 200px;
	overflow-y: auto;
}

.form-control {
	width: 100%;
	border-radius: 5px;
	border: 1px solid white;
	background-color: #333;
	color: white;
	text-transform: uppercase;
	font-weight: bolder;
}

input.form-control {
	width: 93%;
}

.commune-dropdown li {
	padding: 5px 10px;
	cursor: pointer;
}

*:focus {
	outline: none;
}

.commune-dropdown li:hover {
	background-color: #f0f0f0;
}

input,
select,
button {
	font-size: 16px;
	padding: 10px;
}
</style>
