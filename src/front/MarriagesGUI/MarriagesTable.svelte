<script>
	import {
		onMount
	} from "svelte";

	import {
        pop
    } from "svelte-spa-router";

	import Table from "sveltestrap/src/Table.svelte";
	import Button from "sveltestrap/src/Button.svelte";


	import Input from "sveltestrap/src/Input.svelte";
	import Label from "sveltestrap/src/Label.svelte";
	import FormGroup from "sveltestrap/src/FormGroup.svelte";

	import { Pagination, PaginationItem, PaginationLink } from 'sveltestrap';

	let marriages = [];
	let newMarriage = {
		country: "",
		year: parseInt("") ,
		marriages: "",
		avg_wm:"",
		avg_m:""
	};

//Usaremos estas variables para la paginacion y para la busqueda
	let countries = [];
	let years = [];
	let currentCountry = "-";
	let currentYear = "-";

	let numberElementsPages = 10;
	let offset = 0;
	let currentPage = 1; 
	let moreData = true; 

	onMount(getMarriages);
	onMount(getCountriesYears);



	async function ReloadTable() {
		const res = await fetch("/api/v1/global-marriages/loadInitialData")

		if (res.ok) {
			const initialMarriages = await res.json();
			console.log("Contados "+ initialMarriages.length +" datos de matrimonios")
			getMarriages();
		}else{
			console.log("No se han cargado correctamente los datos inicales")
		}
	}



//Funcion que devuelve array con los años y los paises existentes para poder hacer un select y usarlo para buscar
	async function getCountriesYears() {
		const res = await fetch("/api/v1/global-marriages"); //Recogemos los datos de /api/v1/global-marriages

		if (res.ok) {
			const json = await res.json();

			countries = json.map((d) => {
					return d.country;            //Guardamos los paises 
			});
			countries = Array.from(new Set(countries));   //Eliminamos los duplicados
			
			
			years = json.map((d) => {   
					return d.year;    //Guardamos los años en un array
			});
			years = Array.from(new Set(years));      //Eliminamos años repetidos

			console.log("Contados " + countries.length + "paises y " + years.length + "años distintos.");

		} else {
			console.log("ERROR!");
		}
	}

	


	async function getMarriages() {

		console.log("Fetching marriages...");
		const res = await fetch("/api/v1/global-marriages?offset=" + numberElementsPages * offset + "&limit=" + numberElementsPages); 

		if (res.ok) {
			console.log("Ok:");
			const json = await res.json();
			marriages = json;
			console.log("Received " + marriages.length + " marriages.");

			if (marriages.length!=10){
				moreData=false
			} else{

						const next = await fetch("/api/v1/global-marriages?offset=" + numberElementsPages * (offset+1) + "&limit=" + numberElementsPages); 
						console.log("La variable NEXT tiene el estado: " + next.status)
						const jsonNext = await next.json();
						
						
						
						if (jsonNext.length == 0 || next.status==404) {  
							moreData = false;
						} 
						else {
							moreData = true;  //Vemos si quedan aun mas datos en la siguiente pagina
						}
					}
		} 
		else {
			errorAlert("Error al intentar obtener todos los datos");
			console.log("ERROR!");
		}
	}

	async function insertMarriage() {

		console.log("Inserting marriage..." + JSON.stringify(newMarriage));

		if (newMarriage.country == ""
			|| newMarriage.country == null
			|| newMarriage.year == "" 
			|| newMarriage.year == null) {
			
			alert("Se debe incluir el nombre del país y el año obligatoriamente");

		} else {
				const res = await fetch("/api/v1/global-marriages", {
					method: "POST",
					body: JSON.stringify(newMarriage),
					headers: {
						"Content-Type": "application/json"
					}
				}).then(function (res) {
					if (res.ok){
						getMarriages();
					} else{
						errorAlert("Error al intentar insertar el elemento")
					}
					
				});
			}
	}


	//Borramos un pais en un año concreto
	async function deleteMarriage(country,year) {
		console.log("Deleting marrriage..." + JSON.stringify(country)+ + JSON.stringify(year) );

		const res = await fetch("/api/v1/global-marriages/" + country+"/"+year, {
			method: "DELETE"
		}).then(function (res) {
			if (res.ok){
			getMarriages();
			getCountriesYears();
			errorResponse(res)
			} else if (res.status == 404) {
				errorAlert("Se ha intentado borrar un elemento inexistente.");
			} else {
				errorAlert("Error al intentar borrar un elemento concreto");
			}
		});
	}

	//Borramos todos los paises
	async function deleteGlobalMarriages() {
		console.log("Deleting all marriages data...");
		const res = await fetch("/api/v1/global-marriages/", {
			method: "DELETE"
		}).then(function (res) {
			if (res.ok){
			const json =  res.json();
			marriages = json;
		} else{
			errorAlert("Error al tratar de borrar todos los elementos")
		}
			//getMarriages();
			//getCountriesYears();
		});
	}


	async function search(country, year) {
		console.log("Searching data: " + country + " and " + year);

		/* Checking if the fields are empty */
		var url = "/api/v1/global-marriages";

		if (country != "-" && year != "-") {
			url = url + "?country=" + country + "&year=" + year; 
		} else if (country != "-" && year == "-") {
			url = url + "?country=" + country;
		} else if (country == "-" && year != "-") {
			url = url + "?year=" + year;
		}

		const res = await fetch(url);

		if (res.ok) {
			console.log("Ok:");
			const json = await res.json();
			marriages = json;			

			console.log("Found " + marriages.length + " global marrriages stats.");
		} else {
			errorAlert("Error al realizar la busqueda solicitada")
			console.log("ERROR!");
		}
		
	}

	function addOffset (increment) {
		offset += increment;
		currentPage += increment;
		getMarriages();
	}

	/* These functions are for the alerts */ 
	function insertAlert() {
		clearAlert();
		var alert_element = document.getElementById("div_alert");
		alert_element.style = "position: fixed; top: 0px; top: 1%; width: 90%;";
		alert_element.className = "alert alert-dismissible in alert-success ";
		alert_element.innerHTML = "<strong>¡Dato insertado!</strong> El dato ha sido insertado correctamente";
		
		setTimeout(() => {
			clearAlert();
		}, 3000);
	}
	
	function deleteAlert() {
		clearAlert();
		var alert_element = document.getElementById("div_alert");
		alert_element.style = "position: fixed; top: 0px; top: 1%; width: 90%;";
		alert_element.className = "alert alert-dismissible in alert-danger ";
		alert_element.innerHTML = "<strong>¡Dato borrado!</strong> El dato ha sido borrado correctamente";
		
		setTimeout(() => {
			clearAlert();
		}, 3000);
	}

	function deleteAllAlert() {
		clearAlert();
		var alert_element = document.getElementById("div_alert");
		alert_element.style = "position: fixed; top: 0px; top: 1%; width: 90%;";
		alert_element.className = "alert alert-dismissible in alert-danger ";
		alert_element.innerHTML = "<strong>¡Datos borrados!</strong> Todos los datos han sido borrados correctamente";
		
		setTimeout(() => {
			clearAlert();
		}, 3000);
	}

	
	function ReloadTableAlert() {
		clearAlert();
		var alert_element = document.getElementById("div_alert");
		alert_element.style = "position: fixed; top: 0px; top: 1%; width: 90%;";
		alert_element.className = "alert alert-dismissible in alert-danger ";
		alert_element.innerHTML = "<strong>¡Tabla Recargada!</strong> Se han vuelto a los valores iniciales";
		
		setTimeout(() => {
			clearAlert();
		}, 3000);
	}


	function errorAlert(error) {
		clearAlert();
		var alert_element = document.getElementById("div_alert");
		alert_element.style = "position: fixed; top: 0px; top: 1%; width: 90%;";
		alert_element.className = "alert alert-dismissible in alert-danger ";
		alert_element.innerHTML = "<strong>¡ERROR!</strong> ¡Ha ocurrido un error! " + error;
		
		setTimeout(() => {
			clearAlert();
		}, 3000);
	}

	function clearAlert () {
		var alert_element = document.getElementById("div_alert");
		alert_element.style = "display: none; ";
		alert_element.className = "alert alert-dismissible in";
		alert_element.innerHTML = "";
	}

function errorResponse(res) {
	var status = res.status
	switch (status) {
		case 400:
			alert("Codigo de error: " + status + '\n'+ "Error de prueba");
			break;
		case 401:
			alert("Codigo de error: " + status + '\n'+ "Error de prueba 1");
			break;
		case 404:
			alert("Codigo de error: " + status + '\n'+ "Error de prueba 1");
			break;
		case 405:
			alert("Codigo de error: " + status + '\n'+ "Error de prueba 1");
			break;
		case 405:
			alert("Codigo de error: " + status + '\n'+ "Error de prueba 1");
			break;

		default:
			alert("Codigo de error: "+ status +'\n'+ "Error de desconocido")
			break;
	}
}


</script>

<main>
	<div role="alert" id="div_alert" style="display: none;">
	</div>
	{#await marriages}
		Loading marriages...
	{:then marriages}

		<FormGroup> 
			<Label for="selectCountry"> Búsqueda por país </Label>
			<Input type="select" name="selectCountry" id="selectCountry" bind:value="{currentCountry}">
				{#each countries as country}
				<option>{country}</option>
				{/each}
				<option>-</option>
			</Input>
		</FormGroup>
				
		<FormGroup>
			<Label for="selectYear"> Año </Label>
			<Input type="select"  name="selectYear" id="selectYear" bind:value="{currentYear}">
				{#each years as year}
				<option>{year}</option>
				{/each}
				<option>-</option>
			</Input>
		</FormGroup>

		<Button outline color="secondary" on:click="{search(currentCountry, currentYear)}" class="button-search" > <i class="fas fa-search"></i> Buscar </Button>
		
		<Button outline color="primary" on:click="{ReloadTable}"  on:click={ReloadTableAlert}> <i class="fas fa-search"></i> Recargar API </Button>


		<Table bordered>
			<thead>
				<tr>
					<th>Pais</th>
					<th>Año</th>
					<th>Matrimonios Registrados</th>
					<th>Media de edad en hombres</th>
					<th>Media de edad en mujeres</th>
					<th>Actions</th>

				</tr>
			</thead>
			<tbody>
				<tr>
					<td><input bind:value="{newMarriage.country}"></td>
					<td><input type="number" bind:value="{newMarriage.year}"></td>
					<td><input type="number" bind:value="{newMarriage.marriages}"></td>
					<td><input type="number" bind:value="{newMarriage.avg_m}"></td>
					<td><input type="number" bind:value="{newMarriage.avg_wm}"></td>
					<td> <Button outline  color="primary" on:click={insertMarriage}  on:click={insertAlert}> Insertar</Button> </td>
				</tr>
				{#each marriages as marriage}
					<tr>
						<td>
							<a href="#/global-marriages/{marriage.country}/{marriage.year}">{marriage.country}</a>
						</td>
						<td>{marriage.year}</td>
						<td>{marriage.marriages}</td>
						<td>{marriage.avg_m}</td>
						<td>{marriage.avg_wm}</td>
						<td><Button outline color="danger" on:click="{deleteMarriage(marriage.country,marriage.year)}" on:click={deleteAlert}>  <i class="fa fa-trash" aria-hidden="true"></i> Borrar</Button></td>
					</tr>
				{/each}
			</tbody>
		</Table>
	{/await}
 
	<Pagination style="float:right;" ariaLabel="Cambiar de página">


		<PaginationItem class="{currentPage === 1 ? 'disabled' : ''}">
		  <PaginationLink previous href="#/globalMarriagesAPI" on:click="{() => addOffset(-1)}" />
		</PaginationItem>
		
		<!-- If we are not in the first page-->
		{#if currentPage != 1}
		<PaginationItem>
			<PaginationLink href="#/globalMarriagesAPI" on:click="{() => addOffset(-1)}" >{currentPage - 1}</PaginationLink>
		</PaginationItem>
		{/if}
		<PaginationItem active>
			<PaginationLink href="#/globalMarriagesAPI" >{currentPage}</PaginationLink>
		</PaginationItem>

		<!-- If there are more elements-->
		{#if moreData}
		<PaginationItem >
			<PaginationLink href="#/globalMarriagesAPI" on:click="{() => addOffset(1)}">{currentPage + 1}</PaginationLink>
		</PaginationItem>
		{/if}

		<PaginationItem class="{moreData ? '' : 'disabled'}">
		  <PaginationLink next href="#/globalMarriagesAPI" on:click="{() => addOffset(1)}"/>
		</PaginationItem>

	</Pagination>

	<Button outline color="secondary" on:click="{pop}"> <i class="fas fa-arrow-circle-left"></i> Atrás </Button>
	<Button outline on:click={deleteGlobalMarriages}   on:click={deleteAllAlert} color="danger"> <i class="fa fa-trash" aria-hidden="true"></i> Borrar todo </Button>

</main>