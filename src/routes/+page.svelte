<script lang="ts">
	import { onMount } from 'svelte';

	// Variáveis de estado para armazenar os dados das APIs
	let saudacao = 'Carregando saudação...';
	let nomePessoa = 'carregando nome...';

	// Variáveis de estado para os formulários de cadastro
	let novoNome = '';
	let novaSaudacao = '';
	
	// Variável para feedback ao usuário
	let notificacao = { mensagem: '', tipo: '' }; // tipo pode ser 'sucesso' ou 'erro'

	// URLs das APIs
	const urlApiSaudacoes = 'http://localhost:8080/api/saudacoes';
	const urlApiPessoas = 'http://localhost:8000/pessoas';

	// --- FUNÇÕES DE BUSCA DE DADOS ---

	async function buscarSaudacaoAleatoria() {
		try {
			const response = await fetch(`${urlApiSaudacoes}/aleatorio`);
			if (!response.ok) {
				throw new Error('Falha ao buscar saudação');
			}
			const data = await response.json();
			// A API retorna { "saudação": "..." }
			saudacao = data.saudação; 
		} catch (error) {
			console.error('Erro na API de saudações:', error);
			saudacao = 'Oops!';
			mostrarNotificacao('Não foi possível conectar à API de Saudações.', 'erro');
		}
	}

	async function buscarPessoaAleatoria() {
		try {
			const response = await fetch(`${urlApiPessoas}/aleatoria`);
			if (!response.ok) {
				throw new Error('Falha ao buscar pessoa');
			}
			const data = await response.json();
			// A API retorna { "id": ..., "nome": "..." }
			nomePessoa = data.nome;
		} catch (error) {
			console.error('Erro na API de pessoas:', error);
			nomePessoa = 'ninguém';
			mostrarNotificacao('Não foi possível conectar à API de Pessoas.', 'erro');
		}
	}
	
	function buscarTudo() {
		buscarSaudacaoAleatoria();
		buscarPessoaAleatoria();
	}

	// Executa as buscas quando o componente é montado na tela
	onMount(() => {
		buscarTudo();
	});

	// --- FUNÇÕES DE CADASTRO (POST) ---

	async function cadastrarPessoa(event: SubmitEvent) {
		if (!novoNome.trim()) return;

		try {
			const response = await fetch(urlApiPessoas, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({ nome: novoNome })
			});

			if (!response.ok) {
				throw new Error('A resposta da rede não foi OK');
			}
			
			mostrarNotificacao(`Pessoa "${novoNome}" cadastrada com sucesso!`, 'sucesso');
			novoNome = ''; // Limpa o campo
			
			// Opcional: buscar uma nova pessoa para possivelmente mostrar a que acabou de ser cadastrada
			buscarPessoaAleatoria();

		} catch (error) {
			console.error('Erro ao cadastrar pessoa:', error);
			mostrarNotificacao('Erro ao cadastrar nova pessoa.', 'erro');
		}
	}

	async function cadastrarSaudacao(event: SubmitEvent) {
		if (!novaSaudacao.trim()) return;
		
		try {
			const response = await fetch(urlApiSaudacoes, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				// A API espera um objeto com a chave "saudação"
				body: JSON.stringify({ saudação: novaSaudacao })
			});

			if (!response.ok) {
				throw new Error('A resposta da rede não foi OK');
			}
			
			mostrarNotificacao(`Saudação "${novaSaudacao}" cadastrada com sucesso!`, 'sucesso');
			novaSaudacao = ''; // Limpa o campo
			
			// Opcional: buscar uma nova saudação
			buscarSaudacaoAleatoria();
			
		} catch (error) {
			console.error('Erro ao cadastrar saudação:', error);
			mostrarNotificacao('Erro ao cadastrar nova saudação.', 'erro');
		}
	}
	
	function mostrarNotificacao(msg: string, tipoMsg: 'sucesso' | 'erro') {
		notificacao = { mensagem: msg, tipo: tipoMsg };
		setTimeout(() => {
			notificacao = { mensagem: '', tipo: '' };
		}, 4000); // A notificação some depois de 4 segundos
	}

</script>

<main class="min-h-screen bg-gray-100 flex flex-col items-center justify-center p-4 text-gray-800">
	
	{#if notificacao.mensagem}
		<div
			class="fixed top-5 px-6 py-3 rounded-lg text-white font-semibold shadow-lg animate-pulse"
			class:bg-green-500={notificacao.tipo === 'sucesso'}
			class:bg-red-500={notificacao.tipo === 'erro'}
		>
			{notificacao.mensagem}
		</div>
	{/if}

	<div class="bg-white rounded-xl shadow-2xl p-8 md:p-12 text-center w-full max-w-2xl">
		
		<h1 class="text-4xl md:text-6xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-blue-500 to-purple-600">
			{saudacao}, {nomePessoa}!
		</h1>
		
		<button
			on:click={buscarTudo}
			class="mt-6 bg-indigo-500 hover:bg-indigo-600 text-white font-bold py-2 px-6 rounded-full transition-transform transform hover:scale-105"
		>
			Gerar Nova Saudação
		</button>

	</div>

	<div class="mt-12 grid grid-cols-1 md:grid-cols-2 gap-10 w-full max-w-4xl">
		
		<div class="bg-white rounded-xl shadow-2xl p-8">
			<h2 class="text-2xl font-bold mb-4 text-gray-700">Cadastrar Nova Pessoa</h2>
			<form on:submit|preventDefault={cadastrarPessoa} class="flex flex-col gap-4">
				<label for="nome" class="sr-only">Nome da Pessoa</label>
				<input
					id="nome"
					type="text"
					bind:value={novoNome}
					placeholder="Ex: Maria"
					class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
					required
				/>
				<button type="submit" class="w-full bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-lg">
					Cadastrar Pessoa
				</button>
			</form>
		</div>

		<div class="bg-white rounded-xl shadow-2xl p-8">
			<h2 class="text-2xl font-bold mb-4 text-gray-700">Cadastrar Nova Saudação</h2>
			<form on:submit|preventDefault={cadastrarSaudacao} class="flex flex-col gap-4">
				<label for="saudacao" class="sr-only">Texto da Saudação</label>
				<input
					id="saudacao"
					type="text"
					bind:value={novaSaudacao}
					placeholder="Ex: E aí, tudo certo?"
					class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500 focus:outline-none"
					required
				/>
				<button type="submit" class="w-full bg-purple-500 hover:bg-purple-600 text-white font-bold py-2 px-4 rounded-lg">
					Cadastrar Saudação
				</button>
			</form>
		</div>
	</div>

</main>