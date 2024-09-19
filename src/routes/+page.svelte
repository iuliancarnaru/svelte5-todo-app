<script lang="ts">
	type Todo = {
		id: string;
		text: string;
		done: boolean;
	};

	type Filters = 'all' | 'active' | 'completed';

	let todos = $state<Todo[]>([]);
	let filter = $state<Filters>('all');
	let filteredTodos = $derived(filterTodos(todos));

	$effect(() => {
		const savedTodos = localStorage.getItem('todos');
		savedTodos && (todos = JSON.parse(savedTodos));
	});

	$effect(() => {
		localStorage.setItem('todos', JSON.stringify(todos));
	});

	function addTodo(event: KeyboardEvent) {
		if (event.key !== 'Enter') return;
		const inputEl = event.target as HTMLInputElement;

		const todo = {
			id: window.crypto.randomUUID(),
			text: (event.target as HTMLInputElement).value,
			done: false
		};

		todos = [...todos, todo];
		inputEl.value = '';
	}

	function editTodo(event: Event) {
		const inputEl = event.target as HTMLInputElement;
		const index = +inputEl.dataset.index!;
		todos[index].text = inputEl.value;
	}

	function toggleTodo(event: Event) {
		const inputEl = event.target as HTMLInputElement;
		const index = +inputEl.dataset.index!;
		todos[index].done = !todos[index].done;
	}

	function setFilter(newFilter: Filters) {
		filter = newFilter;
	}

	function filterTodos(todos: Todo[]) {
		switch (filter) {
			case 'all':
				return todos;
			case 'active':
				return todos.filter((todo) => !todo.done);
			case 'completed':
				return todos.filter((todo) => todo.done);
			default:
				return todos;
		}
	}

	function remaining() {
		return todos.filter((todo) => !todo.done).length;
	}
</script>

<div class="wrapper">
	<input onkeydown={addTodo} type="text" placeholder="Add a todo..." />

	<div class="todos">
		{#each filteredTodos as todo, i (todo.id)}
			<div class="todo" class:completed={todo.done}>
				<input oninput={editTodo} data-index={i} value={todo.text} type="text" />
				<input onchange={toggleTodo} data-index={i} checked={todo.done} type="checkbox" />
			</div>
		{/each}
	</div>
</div>

<div class="filters">
	{#each ['all', 'active', 'completed'] as filter}
		<button onclick={() => setFilter(filter as Filters)}>{filter}</button>
	{/each}
</div>

<p>{remaining()} remaining</p>

<style lang="scss">
	.wrapper {
		max-width: 900px;
		margin: 0 auto;
	}

	.todos {
		display: grid;
		gap: 1rem;
		margin-block-start: 1rem;
	}

	.todo {
		position: relative;
		transition: all 0.2s ease-in;
	}

	.completed {
		opacity: 0.4;
	}

	input[type='text'] {
		width: 100%;
		padding: 1rem;
	}

	input[type='checkbox'] {
		position: absolute;
		right: 4%;
		top: 50%;
		transform: translateY(-50%);
	}

	.filters {
		margin-top: 20px;
		display: flex;
		gap: 4px;

		& button {
			padding: 7px 15px;
		}
	}
</style>
