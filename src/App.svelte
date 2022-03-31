<script>

import { onMount } from "svelte";
  const endpoint = "http://chat/api/todo";
  let todoList = [];

  onMount(async function () {
    const response = await fetch(endpoint);
    todoList = await response.json();
  });

	let valueInput = '';

	async function deleteElement(id,index){
		const res = await fetch('http://chat/api/todo/delete', {
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

	async function createNewEl(){
		const res = await fetch('http://chat/api/todo/create', {
			method: 'POST',
			body: JSON.stringify({
				valueInput,
				status:'false',
			})
		})
		
		const json = await res.json()
	
		console.log(json);
		
		todoList = [json,...todoList];
		valueInput = '';
	}

	async function changeStatus(index,id){
		todoList[index].status = !todoList[index].status;
		const res = await fetch('http://chat/api/todo/changestatus', {
			method: 'POST',
			body: JSON.stringify({
				id,
				status:todoList[index].status,
			})
		})

	}



</script>

<div class="container mx-auto mt-5">

	<div class="flex justify-center items-center">
		<input type="text" bind:value={valueInput} class="px-3 py-2 rounded border">
		<button class="ml-2 px-2 py-2 bg-black text-white rounded border" on:click={createNewEl}>Create</button>
	</div>

	<div class="w-1/3 mx-auto border rounded px-4 py-4 mt-10">
		<ul>
			{#each todoList as el,index}
			<li class="{el.status == true ? 'active' : ''} flex align-center justify-between px-4 mb-4 py-2 border rounded transition"> 
				<span class="text-lg cursor-pointer hover:text-sky-600" on:click="{()=>changeStatus(index,el.id)}">{el.value}</span>
				<div class="delete px-2 cursor-pointer bg-rose-600 hover:bg-rose-800 transition
					text-white flex justify-center items-center rounded" on:click={()=>deleteElement(el.id,index)}>
					
					<svg width="12" height="12" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path d="M1 1.00006L11 11.0001M1 11.0001L11 1.00006" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
					</svg>
						
				</div>
			</li>
			{/each}
			
		</ul>
	</div>

</div>

<style>
	body {
		font-family: 'Poppins', sans-serif;
	}
	li.active {
		opacity: 0.8;
		background-color: #eee;
		position: relative;
		transition: 0.4s;
	}
</style>