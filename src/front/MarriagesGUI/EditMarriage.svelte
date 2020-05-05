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


    export let params = {};
    let marriage = {};
    let updatedCountry = "XXXX";
    let updatedYear = 12345;
    let updatedMarriages =33333;
    let updatedAvg_m =33;
    let updatedAvg_wm =33;
    let errorMsg = "";

    onMount(getMarriage);

    async function getMarriage() {

        console.log("Fetching marriage...");
        const res = await fetch("/api/v1/global-marriages/" + params.country +"/"+params.year);
        
        if (res.ok) {
            console.log("Ok:");
            const json = await res.json();
            marriage = json;
            updatedCountry = marriage.country;
            updatedYear = marriage.year;
            updatedMarriages = marriage.marriages;
            updatedAvg_m=marriage.avg_m;
            updatedAvg_wm=marriage.avg_wm;
            console.log("Received marriage.");
        } else {
            errorMsg = res.status + ": " + res.statusText;
            console.log("ERROR!" + errorMsg);
        }
    }


    async function updateMarriage() {

        console.log("Updating marriage..." + JSON.stringify(params.country));

        const res = await fetch("/api/v1/global-marriages/" + params.country +"/"+params.year, {
            method: "PUT",
            body: JSON.stringify({
                country: params.country,
                year: parseInt(params.year),
                marriages: updatedMarriages,
                avg_m: updatedAvg_m,
                avg_wm: updatedAvg_wm
            }),
            headers: {
                "Content-Type": "application/json"
            }
        }).then(function (res) {
            if (res.ok) {
                 getMarriage();
            } else if (res.status == 404) {
                    errorAlert("Se ha intentado borrar un elemento inexistente.");
                } else {
                    errorAlert("");
                }
        });
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
    
    function updateAlert() {
		clearAlert();
		var alert_element = document.getElementById("div_alert");
		alert_element.style = "position: fixed; top: 0px; top: 1%; width: 90%;";
		alert_element.className = "alert alert-dismissible in alert-info ";
		alert_element.innerHTML = "<strong>¡Dato actualizado!</strong> El dato ha sido actualizado correctamente";
		
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

    
</script>
<main>

    <div role="alert" id="div_alert" style="display: none;">
	</div>
    <h3>Edit Marriage <strong>{params.country}</strong></h3>
    {#await marriage}
        Loading marriages...
    {:then marriage}
        <Table bordered>
            <thead>
                <tr>
                    <th>Pais</th>
                    <th>Año</th>
                    <th>Matrimonios</th>
                    <th>Media en hombres</th>
                    <th>Media en mujeres</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>{updatedCountry}</td>
                    <td>{updatedYear}</td>
                    <td><input type="number" bind:value="{updatedMarriages}"></td>
                    <td><input type="number" bind:value="{updatedAvg_m}"></td>
                    <td><input type="number" bind:value="{updatedAvg_wm}"></td>
                    <td> <Button outline  color="primary" on:click={updateMarriage} on:click={updateAlert}> <i class="fas fa-pencil-alt"> <i class="fas fa-pencil-alt"> </i> Actualizar</Button> </td>
                </tr>
        </tbody>
        </Table>
    {/await}
    {#if errorMsg}
        <p style="color: red">ERROR: {errorMsg}</p>
    {/if}
    <Button outline color="secondary" on:click="{pop}">Atras</Button>
</main>