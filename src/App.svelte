<script>
	import {
		onMount
	} from "svelte";
	const endpoint = "https://yolly.pro/api/todo";
	let todoList = [];

	onMount(async function () {
		const response = await fetch(endpoint);
		todoList = await response.json();
	});

	let valueInput = '';
	let updateValue = '';
	let updateStatus = false;
	let activeStatus = '';
	let activeIndex = '';
	let updateIndex = '';
	let error = false;
	let errorMsg = '';

	async function deleteElement(id, index) {
		const res = await fetch('https://yolly.pro/api/todo/delete', {
			method: 'POST',
			body: JSON.stringify({
				id
			})
		})

		const json = await res.json()

		console.log(json);

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

		const json = await res.json()

		console.log(json);

		todoList = [json, ...todoList];
		valueInput = '';
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

		if(updateIndex == index){
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
		updateValue = activeIndex = activeStatus = '';
		updateStatus = false;
	}
	
	
	function editElement(index, id) {
		if(updateIndex === index){
			updateStatus = !updateStatus;
			updateValue = activeStatus =  activeIndex = updateIndex = '';
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
		<button class="ml-2 px-2 py-2 bg-emerald-600 hover:bg-emerald-700 text-white rounded border" on:click={createNewEl}>Create</button>
	</div>

	<div class="w-1/3 mx-auto border rounded px-4 py-4 mt-10">

		<div class="{error == false ? 'hidden' : ''} bg-rose-600 rounded flex py-2 my-4 justify-center text-white">
			{errorMsg}
		</div>

		<div class="{updateStatus == false ? 'hidden' : ''} flex justify-center items-center border rounded mb-2 mt-2 py-4 px-4">
			<input type="text" bind:value={updateValue} class="px-3 py-2 rounded border w-full">
			<button class="ml-2 px-2 py-2 bg-green-500 text-white rounded border" on:click="{updateElement(activeIndex , activeStatus)}">Update</button>
		</div>
		<ul>
			{#each todoList as el,index}
			<li class="{el.status == true ? 'active' : ''} {activeStatus == el.id ? 'editmode' : ''} flex align-center justify-between px-4 mb-4 py-2 border rounded transition"> 
				<span class="text-lg cursor-pointer hover:text-sky-600" on:click="{()=>changeStatus(index,el.id)}">
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
				<div class="delete ml-1 px-2 cursor-pointer bg-rose-600 hover:bg-rose-800 transition
					text-white flex justify-center items-center rounded" on:click={()=>deleteElement(el.id,index)}>
					<svg width="12" height="12" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path d="M1 1.00006L11 11.0001M1 11.0001L11 1.00006" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
					</svg>
				</div>
				</div>
			
			</li>
			{/each}
		</ul>
	</div>

</div>

<style>
	li.active {
		opacity: 0.8;
		background-color: #eee;
		position: relative;
		transition: 0.4s;
	}
	.editbutton.active {
		opacity:0.7;
		transform: rotateY(180deg);
	}
	.editmode {
		transition: 0.4s;
		position: relative;
	}
	.editmode::before {
		content: 'editing';
		position: absolute;
		right: calc(100% + 10px);
		font-size: 12px;
		padding: 4px 10px;
		font-weight: 700;
		background-color: rgb(138, 39, 219);
		color: #fff;
		border-radius: 6px;
		text-transform: uppercase;
	}
</style>