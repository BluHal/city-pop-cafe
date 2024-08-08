<script lang="ts">
	import type { Todo } from '$lib/types';
	import { createEventDispatcher, onMount } from 'svelte';
	import { linear } from 'svelte/easing';
	import { fade } from 'svelte/transition';

	const animationOptions = { duration: 100, easing: linear };
	const dispatch = createEventDispatcher();

	let showTodoModal = false;
	let showToolTip = false;
	let todos: Todo[] = [];

	let todoInputValue: string = '';
	let todoInputRef: HTMLInputElement | null = null;

	onMount(() => {
		const storedTodos = localStorage.getItem('todos');
		if (storedTodos) {
			todos = JSON.parse(storedTodos);
		}
	});

	function addTodo(todo: Todo) {
		todos = [todo, ...todos];
		localStorage.setItem('todos', JSON.stringify(todos));
	}

	function removeTodo(index: number) {
		todos = todos.filter((_, i) => i !== index);

		localStorage.setItem('todos', JSON.stringify(todos));
	}

	function updateTodo(index: number) {
		todos[index].done = !todos[index].done;
		localStorage.setItem('todos', JSON.stringify(todos));
	}

	function toggleTodoModal() {
		showTodoModal = !showTodoModal;
		dispatch('todoModalToggle', showTodoModal);
	}

	function onAddTodoEnter(event: KeyboardEvent): void {
		if (event.key === 'Enter') {
			let todo: Todo = {
				name: todoInputValue,
				done: false
			};
			addTodo(todo);
			todoInputValue = '';
			setTimeout(() => {
				todoInputRef?.focus();
			}, 1);
		}
	}
</script>

<div>
	<div class="relative">
		<button
			class="cursor-pointer"
			on:mouseenter={() => (showToolTip = !showToolTip)}
			on:mouseleave={() => (showToolTip = !showToolTip)}
			on:click={() => toggleTodoModal()}
		>
			<i class="fa-solid fa-clipboard-list fa-lg shadow-icon"></i>
		</button>

		{#if showToolTip}
			<div
				class="absolute -left-6 top-8 p-1 rounded-lg shadow-tooltip z-50"
				transition:fade={animationOptions}
			>
				<span class="shadow-text text-sm text-nowrap">To Do</span>
			</div>
		{/if}
	</div>

	{#if showTodoModal}
		<div
			class="w-dvw h-dvh fixed inset-0 bg-white bg-opacity-10 backdrop-blur-sm flex justify-center items-center z-[100]"
		>
			<div
				class="relative w-3/4 md:2/3 lg:w-2/5 bg-black bg-opacity-90 flex flex-col justify-center text-center rounded-2xl p-4 md:p-8"
			>
				<div class="absolute text-white top-0 right-0 m-3">
					<button class="cursor-pointer" on:click={() => toggleTodoModal()}
						><i class="fa-solid fa-xmark fa-lg shadow-icon"></i></button
					>
				</div>
				<span class="text-2xl mb-5">To Do<span class="text-sm">s</span></span>
				<form>
					<input
						type="text"
						class="text-white text-lg rounded w-full md:w-4/5 h-14 placeholder:italic bg-transparent border border-neon-pink px-2 todoInput"
						placeholder="Write a new task"
						on:keydown={onAddTodoEnter}
						bind:this={todoInputRef}
						bind:value={todoInputValue}
					/>
				</form>
				<div class="flex flex-col max-h-96 overflow-y-auto items-center">
					{#each todos as todo, index}
						<div
							class="w-full md:w-4/5 min-h-14 border border-neon-pink rounded py-1 px-3 flex items-center justify-between"
						>
							<div class="flex gap-3 items-center w-5/6">
								<div
									class="p-1 h-6 w-6 border border-white rounded-full cursor-pointer flex items-center justify-center"
								>
									<button
										class="w-full h-full flex items-center justify-center"
										on:click={() => updateTodo(index)}
									>
										{#if todo.done}
											<i class="fa-solid fa-check text-green-700"></i>
										{/if}
									</button>
								</div>
								<span
									class="{todo.done
										? 'line-through decoration-2 text-gray-400 decoration-gray-400'
										: ''} text-nowrap overflow-hidden text-ellipsis w-5/6 text-left"
									>{todo.name}</span
								>
							</div>
							<button class="cursor-pointer" on:click={() => removeTodo(index)}
								><i class="fa-solid fa-trash-can"></i></button
							>
						</div>
					{/each}
				</div>
			</div>
		</div>
	{/if}
</div>

<style>
	.todoInput:focus {
		filter: drop-shadow(0px 0px 2px #ff00ff) drop-shadow(0px 0px 8px #ff00ff);
	}

	.fa-solid:hover {
		filter: drop-shadow(0px 0px 2px #ff00ff) drop-shadow(0px 0px 8px #ff00ff);
	}
</style>
