<script lang="ts">
	import Frame from '$lib/components/Frame.svelte';
	import {
		Tile,
		Checkbox,
		Button,
		RadioButtonGroup,
		RadioButton,
		Slider,
		Tag,
		InlineNotification
	} from 'carbon-components-svelte';
	import { writable, derived } from 'svelte/store';

	import CopyButton from '$lib/components/CopyButton.svelte';
	import Row from '$lib/components/Row.svelte';
	import Column from '$lib/components/Column.svelte';

	const checked_values_order = ['special', 'numbers', 'lowercase', 'uppercase'];

	let checked_values = writable<string[]>([]);
	let checked_values_ordered = derived(checked_values, ($checked_values) =>
		[...$checked_values].sort(
			(a, b) => checked_values_order.indexOf(a) - checked_values_order.indexOf(b)
		)
	);

	let password_length = writable<number>(8);

	let passwords = writable<string[]>([]);

	let is_generation_allowed_charset = derived(checked_values, ($checked_values) => {
		return $checked_values.length > 0;
	});

	let is_generation_allowed_length = derived(
		[password_length, checked_values],
		([$password_length, $checked_values]) => {
			return $password_length > $checked_values.length - 1;
		}
	);

	let is_generation_allowed = derived(
		[is_generation_allowed_charset, is_generation_allowed_length],
		([$is_generation_allowed_charset, $is_generation_allowed_length]) =>
			$is_generation_allowed_charset && $is_generation_allowed_length
	);

	// Observe any changes to reset password array
	$: {
		$checked_values, $password_length;
		passwords.set([]);
	}

	$: {
		if ($password_length < $checked_values.length) {
			$password_length = $checked_values.length;
		}
	}
	$checked_values = ['lowercase', 'uppercase', 'numbers', 'special'];

	async function generate_passwords() {
		let has_symbols = $checked_values.includes('special');
		let has_numbers = $checked_values.includes('numbers');
		let has_lowercase = $checked_values.includes('lowercase');
		let has_uppercase = $checked_values.includes('uppercase');

		/*Сбрасываем список паролей*/
		passwords.set([]);
		/*Генерируем пароли*/
		for (let i = 0; i < 5; i++) {
			/*Генерируем пароль на основе переданных параметров*/
			let password = generate_passwords_extended(
				$password_length,
				has_symbols,
				has_numbers,
				has_uppercase,
				has_lowercase
			);

			/*Добавляем новый пароль в список*/
			$passwords = [...$passwords, password];
			console.log(password);
		}
		console.log($passwords.length);
	}

	const SPECIAL_SYMBOLS_EXTENDED = '!"#$%&\'()*+,-./:;<=>?@[]^_`{|}~';
	const NUMBERS = '0123456789';
	const LOWERCASE = 'abcdefghijklmnopqrstuvwxyz';
	const UPPERCASE = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
	const SPECIAL_SYMBOLS_BASIC = '!@#$%^&*_?';

	function generate_passwords_extended(
		length: number,
		special_symbols: boolean,
		numbers: boolean,
		uppercase: boolean,
		lowercase: boolean
	): string {
		let password: string = '';
		let charset: string[] = [];

		// Проверяем, что хотя бы один из аргументов имеет значение true
		if (!special_symbols && !numbers && !uppercase && !lowercase) {
			// Если ни один из аргументов не имеет значение true, выводим предупреждение и возвращаем пустую строку
			//panic!("Должен быть выбран хотя бы один из аргументов");

			return '';
		}

		if (special_symbols) {
			charset.push(SPECIAL_SYMBOLS_BASIC);
		}

		if (numbers) {
			charset.push(NUMBERS);
		}

		if (uppercase) {
			charset.push(UPPERCASE);
		}

		if (lowercase) {
			charset.push(LOWERCASE);
		}
		console.log('длина пароля ' + length);
		console.log('кол-во наборов ' + charset.length);
		// Проверяем, достаточно ли длины для генерации пароля с учетом выбранных наборов символов
		if (charset.length > length) {
			console.log('длина пароля ' + length);
			console.log('кол-во наборов ' + charset.length);
			console.log('Not enough length');
			return '';
		}
		charset = shuffle(charset);

		// Разделяем длину на количество наборов символов
		let password_chunk_lenght = length / charset.length;
		password_chunk_lenght = Math.floor(password_chunk_lenght);

		// Остаток длины
		let rest_chunk_length = length % charset.length;

		console.log('размер куска пароля ' + password_chunk_lenght);
		console.log('остаток куска пароля ' + rest_chunk_length);

		// Генерируем пароль
		for (const element of charset) {
			for (let i = 0; i < password_chunk_lenght; i++) {
				let random_index = Math.floor(Math.random() * element.length);
				password = [...password, element[random_index]].join('');
			}
		}

		// Генерируем остаток пароля

		for (let i = 0; i < rest_chunk_length; i++) {
			let random_index = Math.floor(Math.random() * charset[0].length);
			password = [...password, charset[0][random_index]].join('');
		}

		// Перемешиваем пароль
		password = shuffle([...password]).join('');

		return password;
	}
	// Перемешиваем наборы символов
	function shuffle(array: string[]) {
		let m = array.length,
			t,
			i;

		// Пока есть элементы для перемешивания
		while (m) {
			// Взять оставшийся элемент
			i = Math.floor(Math.random() * m--);

			// И поменять его местами с текущим элементом
			t = array[m];
			array[m] = array[i];
			array[i] = t;
		}

		return array;
	}

	function get_tag_color(value: string): 'red' | 'blue' | 'gray' | 'warm-gray' | undefined {
		if (value === 'special') {
			return 'red';
		} else if (value === 'numbers') {
			return 'blue';
		} else if (value === 'lowercase') {
			return 'warm-gray';
		} else if (value === 'uppercase') {
			return 'gray';
		}

		return undefined;
	}
</script>

<div class="page-body">
	<div class="page">
		<Column>
			<!-- <div class="tile-space" /> -->
			<Tile light>
				<p>Настройки</p>
				<Row gap={32}>
					<RadioButtonGroup
						legendText="Predefined password lengths"
						name="length"
						bind:selected={$password_length}
					>
						<RadioButton labelText="5" value={5} />
						<RadioButton labelText="8" value={8} />
						<RadioButton labelText="13" value={13} />
						<RadioButton labelText="21" value={21} />
					</RadioButtonGroup>
					<Slider
						labelText="Custom password length"
						min={$checked_values.length}
						max={55}
						value={$password_length}
						on:change={(event) => {
							let value = event.detail;
							// console.log(value);
							$password_length = value;
						}}
					/>
				</Row>
			</Tile>
			<Tile>
				<Row>
					<Checkbox
						bind:group={$checked_values}
						labelText="Special symbols"
						value="special"
						checked
					/>
					<Checkbox bind:group={$checked_values} labelText="Numbers" value="numbers" checked />
					<Checkbox bind:group={$checked_values} labelText="Lowercase" value="lowercase" checked />
					<Checkbox bind:group={$checked_values} labelText="Uppercase" value="uppercase" checked />
				</Row>
				<Row>
					<p class="tags-label">Checked values:</p>
					{#each $checked_values_ordered as value}
						<Tag type={get_tag_color(value)}>{value}</Tag>
					{/each}
				</Row>
			</Tile>

			{#if !$is_generation_allowed}
				<Tile light>
					{#if !$is_generation_allowed_length}
						<InlineNotification
							lowContrast
							hideCloseButton
							kind="warning"
							title="Not enough length:"
							subtitle="Please select a length higher than checked values"
						/>
					{/if}
					{#if !$is_generation_allowed_charset}
						<InlineNotification
							lowContrast
							hideCloseButton
							kind="warning"
							title="No characters selected:"
							subtitle="Please select at least one character option"
						/>
					{/if}
				</Tile>
			{/if}

			<Tile light>
				<Button disabled={!$is_generation_allowed} on:click={generate_passwords}
					>Generate password</Button
				>
			</Tile>

			{#if $passwords.length > 0}
				<Tile light>
					<p>Results go here</p>
					<Column>
						{#if $passwords.length === 0}
							<p>No passwords generated</p>
						{:else}
							{#each $passwords as password}
								<div class="horizontal">
									<CopyButton valueToCopy={password} />
									<p class="password-result">{password}</p>
								</div>
							{/each}
						{/if}
					</Column>
				</Tile>
			{/if}
		</Column>
	</div>
</div>

<style>
	.page-body {
		display: flex;
		width: 100%;
		flex-direction: column;
	}
	.page {
		display: flex;
		align-self: center;
	}

	.password-result {
		font-family: 'IBM Plex Mono', monospace;
	}

	.horizontal {
		display: flex;
		flex-direction: row;
		align-items: center;
	}

	.tags-label {
		margin-right: var(--cds-spacing-03);
		margin-top: var(--cds-spacing-02);
		margin-bottom: var(--cds-spacing-02);
	}
</style>
