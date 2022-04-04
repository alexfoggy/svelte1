<script>
	import {
		onMount
	} from "svelte";
	const endpoint = "https://yolly.pro/api/todo";
	

	let seeMore = false;
	$: limit = 6;
	$: todoListGot = [];
	$: todoList = todoListGot.slice(0,limit);
	$: seeMoreCheck = function(){
		if(todoListGot.length > limit){
			seeMore = true;
		}
		else {
			seeMore = false;
		}
	}()

	onMount(async function () {
		const response = await fetch(endpoint);
		todoListGot = await response.json();
		loading = false;
	});
	let searchInput = '';
	let loading = true;
	let valueInput = null;
	let updateValue = null;
	let updateStatus = false;
	let activeStatus = null;
	let activeIndex = null;
	let updateIndex = null;
	let error = false;
	let errorMsg = null;
	let countDone = 0;
	let countNotDone = 0;

	let commentIndex = '';
	let commentStatus = false;
	let commentValue = '';
	let commentId = '';
	let loadingComments = false;
	$: comments = [];

	$: count = todoListGot.length;

	$: checkIt = function(){
		countDone = countNotDone = 0;
		todoListGot.forEach(function(e){
			if(e.status == 1){
				countDone++;
			}
			else {
				countNotDone++;
			}
		})
	}();


	async function deleteElement(id, index) {
		const res = await fetch('https://yolly.pro/api/todo/delete', {
			method: 'POST',
			body: JSON.stringify({
				id
			})
		})

		const json = await res.json()

		todoListGot.splice(index, 1)
		todoListGot = todoListGot;
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

		todoListGot = [json, ...todoListGot];
		updateValue = activeStatus =  activeIndex = updateIndex = valueInput = '';
		limit++;
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
		 todoListGot[index].status = !todoList[index].status;
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

	function search(){
		if(searchInput === ''){
			todoList = todoListGot.slice(0,limit);
		}
		else {
			todoList = todoListGot.filter(elem => elem.value.toLowerCase().indexOf(searchInput.toLowerCase()) !== -1);
		}
	}

	async function createComment(id){
		if(commentValue != ''){
		const res = await fetch('https://yolly.pro/api/todo/comment/create', {
			method: 'POST',
			body: JSON.stringify({
				commentValue,
				id,
			})
		})

		const json = await res.json();
		
		comments = [json, ...comments];
		commentValue= '';
	}

}

async function openComments(id,index){
		
		if(id != commentIndex){

		commentIndex = id;
		commentStatus = true;
		
		loadingComments = true;
		
		const response = await fetch('https://yolly.pro/api/todo/comment/show/'+id);
		comments = await response.json();

		loadingComments = false;

		commentId = id;
		}
	}
  </script>
   

<div class="container mx-auto mt-5">

	<div class="flex justify-center items-center">
		<div class="">
		<div class="flex justify-between">
		<p class="text-xs mb-2 uppercase font-bold">Search</p>
		<a href="#" on:click={()=>{searchInput = '';search()}} class='text-xs hover:no-underline hover:font-bold duration-500'>Clear</a>
		</div>
		<input type="text" placeholder="type to search..." bind:value={searchInput} on:keyup="{search}" class="px-3 py-2 rounded border w-full mb-4">
		<div class="border rounded px-4 py-2 mb-8">
			<p class="text-xs mb-2 uppercase font-bold">Statistic</p>
			<p class="py-2 text-sm">List length: {count}</p>
			<p class="py-2 text-sm">Done tasks: {countDone}</p>
			<p class="py-2 text-sm">Not finished tasks: {countNotDone}</p>
		</div>
		<input type="text" bind:value={valueInput} placeholder="Type here...." class="px-3 py-2 rounded border">
		<button class="ml-2 px-2 py-2 bg-emerald-500 hover:bg-emerald-600 text-white rounded border" on:click={createNewEl}>Create</button>
		</div>
			
	</div>
	<div class="container mx-auto flex items-start justify-center">
		<div class="w-1/3 border rounded px-4 py-4 mt-10">
		<div class="{error == false ? 'hidden' : ''} bg-rose-600 rounded flex py-2 my-4 justify-center text-white">
			{errorMsg}
		</div>

		<div class="{updateStatus == false ? 'hidden' : ''} flex justify-center items-center border rounded mb-2 mt-2 py-4 px-4">
			<input type="text" bind:value={updateValue} class="px-3 py-2 rounded border w-full">
			<button class="ml-2 px-2 py-2 bg-green-500 text-white rounded border" on:click="{updateElement(activeIndex , activeStatus)}">Update</button>
		</div>
		<ul>
			{#if todoList != ''}
			<p class="text-xs mb-2 uppercase font-bold">List</p>
			{#each todoList as el,index}
			<li class="{el.status == true ? 'active' : ''} {activeStatus == el.id ? 'editmode' : ''} flex items-center justify-between px-4 mb-4 py-2 border rounded-lg transition"> 
				<span class="text-lg cursor-pointer hover:text-sky-600 pr-4" on:click="{()=>changeStatus(index,el.id)}">
					{el.value}
				</span>
				<div class="flex">
						<div class="{commentIndex == el.id ? 'active' : ''} chatopen ml-1 px-2 py-2 cursor-pointer bg-green-500 hover:bg-green-600 transition
						text-white flex justify-center items-center rounded" on:click={()=>openComments(el.id,index)}>
						<svg width="14" height="14" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
							<path d="M10 19C14.9706 19 19 14.9706 19 10C19 5.02944 14.9706 1 10 1C5.02944 1 1 5.02944 1 10C1 11.4876 1.36093 12.891 2 14.1272L1 19L5.8728 18C7.10904 18.6391 8.51237 19 10 19Z" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
						</svg>
						</div>
				
					<div class="{el.status == true ? 'hidden' : ''} {activeStatus == el.id ? 'active' : ''} ml-1 editbutton px-1 cursor-pointer bg-blue-600 hover:bg-blue-800 transition
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
		{#if seeMore == true}
		<div class="flex justify-center">
			<span on:click="{()=>{limit += 3}}" class="text-white bg-emerald-500 rounded px-3 cursor-pointer hover:bg-emerald-600 duration-500 py-1">
				See more
			</span>
		</div>
		{/if}
	</div>
	{#if commentStatus == true}
	<div class="border rounded ml-10 mt-10 px-5 py-3 relative">
		<div class="cursor-pointer hover:opacity-50 absolute top-1.5 right-1.5" on:click={()=>{commentStatus = !commentStatus;commentIndex = 0}}>
			<svg width="12" height="12" viewBox="0 0 6 6" fill="none" xmlns="http://www.w3.org/2000/svg">
			<path d="M1.00024 1L5.00024 5M1.00024 5L5.00024 1" stroke="#292929" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
			</svg>
			</div>
		<p class="text-xs mb-2 uppercase font-bold">Comment</p>
		<div class="my-3">
			{#if comments != ''}
			{#each comments as el,index}
			<div class="border rounded px-4 py-2 mb-2">
				<p class="text-sm">{el.comment}</p>
				<p class="text-xs font-bold mt-2">{	new Date(el.created_at).toLocaleString().split(',')[0]}</p>	
			</div>
			{/each}
			{:else}
			{#if loadingComments == true}
			<p>Loading...</p>
			{:else}
			<p>No comments yet</p>
			{/if}
			{/if}
		</div>
		<div class="flex flex-col">
			<input type="text" class="px-3 py-2 rounded border w-full mb-2" placeholder="write comment here...." bind:value={commentValue}>
			<button on:click={()=>createComment(commentId)} class='mt-2 rounded py-2 text-white bg-emerald-500'>Create comment</button>
		</div>
	</div>
	{/if}
</div>
</div>


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
	.chatopen.active {
		opacity:0.7;
		transform: rotateY(180deg);
	}
</style>