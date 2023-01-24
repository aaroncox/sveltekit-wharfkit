<script lang="ts">
	import welcome from '$lib/images/svelte-welcome.webp';
	import welcome_fallback from '$lib/images/svelte-welcome.png';

	import SessionKit, { Session, WalletPluginPrivateKey } from '@wharfkit/session';

	import WharfKitWeb from '@wharfkit/web';
	console.log(WharfKitWeb);

	let session: Session | undefined;
	let sessionKit: SessionKit = new SessionKit({
		appName: 'demo-app',
		chains: [
			{
				id: '73e4385a2708e6d7048834fbc1079f2fabb17b3c125b146af438971e90716c4d',
				url: 'https://jungle4.greymass.com'
			}
		],
		ui: new WharfKitWeb(),
		walletPlugins: [
			new WalletPluginPrivateKey({
				privateKey: '5Jtoxgny5tT7NiNFp1MLogviuPJ9NniWjnU4wKzaX4t7pL4kJ8s'
			})
		]
	});
	let transactionResult: string;

	async function login() {
		session = await sessionKit.login({ permissionLevel: 'wharfkittest@test' });
	}

	// logout and remove session from storage
	function logout() {
		session = undefined;
	}

	// transfer tokens using a session
	function transact() {
		console.log('transact');
		if (!session) {
			return;
		}
		const action = {
			account: 'eosio.token',
			name: 'transfer',
			authorization: [session.permissionLevel],
			data: {
				from: session.actor,
				to: 'teamgreymass',
				quantity: '0.0001 EOS',
				memo: 'Anchor is the best! Thank you <3'
			}
		};
		session.transact({ action }, { broadcast: false }).then((result) => {
			if (result.resolved) {
				transactionResult = `Transaction broadcast! ${result.resolved.transaction.id}\n`;
			}
		});
	}
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<section>
	<h1>
		<span class="welcome">
			<picture>
				<source srcset={welcome} type="image/webp" />
				<img src={welcome_fallback} alt="Welcome" />
			</picture>
		</span>

		to your new<br />SvelteKit app
	</h1>

	<h2>
		try editing <strong>src/routes/+page.svelte</strong>
	</h2>

	{#if session}
		<p>Logged in as {session.actor}</p>
		<button on:click={transact}>Test Transfer</button>
		<button on:click={logout}>Logout</button>
	{:else}
		<button on:click={login}>Login</button>
	{/if}
</section>

<style>
	button {
		font-size: 30px;
	}
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
	}

	h1 {
		width: 100%;
	}

	.welcome {
		display: block;
		position: relative;
		width: 100%;
		height: 0;
		padding: 0 0 calc(100% * 495 / 2048) 0;
	}

	.welcome img {
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		display: block;
	}
</style>
