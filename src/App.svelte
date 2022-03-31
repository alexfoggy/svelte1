<script>

	let valueInput = '';

	let toDoDefault = [{
			value: 'First to do task',
			status: true
		},
		{
			value: 'Second to do task',
			status: false
		},
		{
			value: 'Third to do task',
			status: true
		},
	]


	function deleteElement(index){
		toDoDefault.splice(index, 1)
		toDoDefault = toDoDefault;
	}

	function createNewEl(){
		toDoDefault = [...toDoDefault, {value: valueInput, status: false}];
		valueInput = '';
	}

</script>

<div class="container mx-auto mt-5">

	<div class="flex justify-center items-center">
		<input type="text" bind:value={valueInput} class="px-3 py-2 rounded border">
		<button class="ml-2 px-2 py-2 bg-black text-white rounded border" on:click={createNewEl}>Create</button>
	</div>

	<div class="w-1/3 mx-auto border rounded px-4 py-4 mt-10">
		<ul>
			{#each toDoDefault as el,index}
			<li class="flex align-center justify-between px-4 mb-4 py-2 border rounded transition"
			 class:active = {el.status}> 
				<span class="text-lg cursor-pointer hover:text-sky-600" on:click="{()=> el.status = !el.status}">{el.value}</span>
				<div class="delete px-2 cursor-pointer bg-rose-600 hover:bg-rose-800 transition
					text-white flex justify-center items-center rounded" on:click={()=>deleteElement(index)}>
					
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
		margin-left: 30px;
		transition: 0.4s;
	}
	li.active::before {
		content: '';
		position: absolute;
		right: calc(100% + 15px);
		top: 50%;
		transform: translateY(-50%);
		width: 10px;
		height: 10px;
		background-color: rgb(42, 161, 42);
		border-radius: 50%;
		transition: 0.4s;
	}
</style>