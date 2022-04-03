<script>
	import {
		onMount
	} from "svelte";
	const endpoint = "https://yolly.pro/api/todo";
	let todoList = [];

	onMount(async function () {
		const response = await fetch(endpoint);
		todoList = await response.json();
		loading = false;
	});
	let loading = true;
	let valueInput = null;
	let updateValue = null;
	let updateStatus = false;
	let activeStatus = null;
	let activeIndex = null;
	let updateIndex = null;
	let error = false;
	let errorMsg = null;

	$: count = todoList.length;


	async function deleteElement(id, index) {
		const res = await fetch('https://yolly.pro/api/todo/delete', {
			method: 'POST',
			body: JSON.stringify({
				id
			})
		})

		const json = await res.json()

		todoList.splice(index, 1)
		todoList = todoList;
	}

	async function createNewEl() {
		if(valueInput != ''){
		const res = await fetch('https://yolly.pro/api/todo/create', {
			method: 'POST',
			body: JSON.stringify({
				valueInput,
				status: 'false',
			})
		})

		const json = await res.json();

		todoList = [json, ...todoList];
		updateValue = activeStatus =  activeIndex = updateIndex = valueInput = '';

		}
		else {
			error = true;
			errorMsg = 'Cannot be empty';
			setTimeout(() => {
				error = false
			}, 3000);
		}
	}

	async function changeStatus(index, id) {
		if(updateIndex === index){
			error = true;
			errorMsg = 'Cannot check task while its in change mode';
			setTimeout(() => {
				error = false
			}, 3000);
	}
	else {
		todoList[index].status = !todoList[index].status;
		const res = await fetch('https://yolly.pro/api/todo/changestatus', {
			method: 'POST',
			body: JSON.stringify({
				id,
				status: todoList[index].status,
			})
		})
		error = false;
		updateIndex = null;
	}

	}

	async function updateElement(index, id) {
		const res = await fetch('https://yolly.pro/api/todo/update', {
			method: 'POST',
			body: JSON.stringify({
				id,
				updateValue,
			})
		})
		todoList[index].value = updateValue;
		updateValue = activeIndex = activeStatus = updateIndex = null;
		updateStatus = false;
	}
	
	
	function editElement(index, id) {
		if(updateIndex === index){
			updateStatus = !updateStatus;
			updateValue = activeStatus = activeIndex = updateIndex = null;
		}
		else {
			activeStatus = id;
			activeIndex = index;
			updateStatus = true;
			updateValue = todoList[index].value;
			updateIndex = index;
		}
	}

  </script>
   

<div class="container mx-auto mt-5">

	<div class="flex justify-center items-center">
		<input type="text" bind:value={valueInput} placeholder="Type here...." class="px-3 py-2 rounded border">
		<button class="ml-2 px-2 py-2 bg-emerald-500 hover:bg-emerald-600 text-white rounded border" on:click={createNewEl}>Create</button>
	</div>

	<div class="w-1/3 mx-auto border rounded px-4 py-4 mt-10">
		<p class="py-2 text-sm">List length: {count	}</p>
		<div class="{error == false ? 'hidden' : ''} bg-rose-600 rounded flex py-2 my-4 justify-center text-white">
			{errorMsg}
		</div>

		<div class="{updateStatus == false ? 'hidden' : ''} flex justify-center items-center border rounded mb-2 mt-2 py-4 px-4">
			<input type="text" bind:value={updateValue} class="px-3 py-2 rounded border w-full">
			<button class="ml-2 px-2 py-2 bg-green-500 text-white rounded border" on:click="{updateElement(activeIndex , activeStatus)}">Update</button>
		</div>
		<ul>
			{#if todoList != ''}
			{#each todoList as el,index}
			<li class="{el.status == true ? 'active' : ''} {activeStatus == el.id ? 'editmode' : ''} flex items-center justify-between px-4 mb-4 py-2 border rounded-lg transition"> 
				<span class="text-lg cursor-pointer hover:text-sky-600 pr-4" on:click="{()=>changeStatus(index,el.id)}">
					{el.value}
				</span>
				<div class="flex">
					<div class="{el.status == true ? 'hidden' : ''} {activeStatus == el.id ? 'active' : ''} editbutton px-1 cursor-pointer bg-blue-600 hover:bg-blue-800 transition
					text-white flex justify-center items-center rounded" on:click={(e)=>editElement(index , el.id)}>
					<svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path d="M5 16L4 20L8 19L19.5858 7.41421C20.3668 6.63316 20.3668 5.36683 19.5858 4.58579L19.4142 4.41421C18.6332 3.63316 17.3668 3.63317 16.5858 4.41421L5 16Z" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
						<path d="M15 6L18 9" stroke="#fff" stroke-width="1" stroke-linecap="round" stroke-linejoin="round"/>
						</svg>
				</div>
				<div class="delete ml-1 px-2 py-2 cursor-pointer bg-rose-600 hover:bg-rose-800 transition
					text-white flex justify-center items-center rounded" on:click={()=>deleteElement(el.id,index)}>
					<svg width="12" height="12" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path d="M1 1.00006L11 11.0001M1 11.0001L11 1.00006" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
					</svg>
				</div>
				</div>
			
			</li>
			{/each}
			{:else}
				{#if loading}
					<p class="text-center">Loading...</p>
				{:else}
					<p class="text-center">todo is empty :(</p>
				{/if}
			{/if}
		</ul>
	</div>

</div>
<link rel="stylesheet" href="/smui.css" />
<style>
	li.active {
		opacity: 0.8;
		background-color: rgb(224, 224, 224);
		transition: 1s;
	}
	li {
		position: relative;
	}
	li::before {
		content: 'in work';
		white-space: nowrap;
		position: absolute;
		right: calc(100% + 10px);
		top: 50%;
		transform: translateY(-50%);
		font-size: 12px;
		padding: 2px 6px;
		font-weight: 700;
		background-color: rgb(25, 91, 177);
		color: #fff;
		border-radius: 6px;
		transition: 1s;
		text-transform: uppercase;
	}
	li.active::before {
		content: 'done';
		background-color: rgb(30, 175, 74);
	}
	.editbutton.active {
		opacity:0.7;
		transform: rotateY(180deg);
	}
	.editmode {
		transition: 0.4s;
		position: relative;
		border: 1px rgb(138, 39, 219) solid;
	}
	.editmode::before {
		content: 'editing';
		background-color: rgb(138, 39, 219);
	}
</style>