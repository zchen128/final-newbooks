<script>
	// Safe destructuring with a default fallback
	let { data = { transactions: [] } } = $props();

	// Safely wrap the array in state with a fallback array
	let transactions = $state(data?.transactions || []);

	// Track active filter mode ('all', 'Revenue', or 'Expense')
	let activeFilter = $state('all');

	// Classification helper
	function classify(t) {
		if (!t) return 'Other';
		if (t.credit === 'Revenue' || t.credit === 'Consulting' || t.credit === 'Sales') {
			return 'Revenue';
		} else if (t.debit && t.debit.includes('Expense')) {
			return 'Expense';
		} else {
			return 'Other';
		}
	}

	// Compute filtered list reactively based on filter state choice
	let filteredTransactions = $derived(
		(transactions || []).filter((t) => {
			if (activeFilter === 'all') return true;
			return classify(t) === activeFilter;
		})
	);

	// Groups revenues into a key-value object: { "Consulting": 500, "Sales": 200 }
	let revenueAccounts = $derived(
		(transactions || [])
			.filter((t) => classify(t) === 'Revenue')
			.reduce((acc, t) => {
				const accountName = t.credit || 'Unclassified Revenue';
				acc[accountName] = (acc[accountName] || 0) + Number(t.amount || 0);
				return acc;
			}, {})
	);

	// Groups expenses into a key-value object: { "Rent Expense": 1000, "Utilities Expense": 150 }
	let expenseAccounts = $derived(
		(transactions || [])
			.filter((t) => classify(t) === 'Expense')
			.reduce((acc, t) => {
				const accountName = t.debit || 'Unclassified Expense';
				acc[accountName] = (acc[accountName] || 0) + Number(t.amount || 0);
				return acc;
			}, {})
	);

	// Dynamic grand sums used to compute total Net Income balance
	let totalRevenue = $derived(Object.values(revenueAccounts).reduce((sum, val) => sum + val, 0));
	let totalExpenses = $derived(Object.values(expenseAccounts).reduce((sum, val) => sum + val, 0));
	let netIncome = $derived(totalRevenue - totalExpenses);
</script>

<div class="mx-auto max-w-5xl space-y-8 p-6">
	<header class="border-b border-slate-200 pb-4">
		<h1 class="text-3xl font-bold text-slate-800">📒 Final New Books</h1>
		<p class="mt-1 text-sm text-slate-500">Track transactions. See your income statement live.</p>
	</header>

	<section class="rounded-lg border border-slate-200 bg-white p-6 shadow-sm">
		<h2 class="mb-4 text-xl font-bold text-slate-800">New Transaction</h2>

		<form method="POST" action="?/create" class="grid grid-cols-1 gap-4 md:grid-cols-2">
			<div>
				<label for="date" class="mb-1 block text-sm font-medium text-slate-700">Date</label>
				<input
					id="date"
					name="date"
					type="date"
					required
					class="w-full rounded border border-slate-300 px-3 py-2"
				/>
			</div>

			<div>
				<label for="amount" class="mb-1 block text-sm font-medium text-slate-700">Amount</label>
				<input
					id="amount"
					name="amount"
					type="number"
					step="0.01"
					placeholder="0.00"
					required
					class="w-full rounded border border-slate-300 px-3 py-2"
				/>
			</div>

			<div class="md:col-span-2">
				<label for="description" class="mb-1 block text-sm font-medium text-slate-700"
					>Description</label
				>
				<input
					id="description"
					name="description"
					type="text"
					placeholder="e.g. Office rent for July"
					required
					class="w-full rounded border border-slate-300 px-3 py-2"
				/>
			</div>

			<div>
				<label for="debit" class="mb-1 block text-sm font-medium text-slate-700"
					>Debit Account</label
				>
				<select
					id="debit"
					name="debit"
					required
					class="w-full rounded border border-slate-300 px-3 py-2"
				>
					<option value="">-- Select --</option>
					<option>Cash</option>
					<option>Accounts Receivable</option>
					<option>Revenue</option>
					<option>Rent Expense</option>
					<option>Utilities Expense</option>
					<option>Owner's Equity</option>
				</select>
			</div>

			<div>
				<label for="credit" class="mb-1 block text-sm font-medium text-slate-700"
					>Credit Account</label
				>
				<select
					id="credit"
					name="credit"
					required
					class="w-full rounded border border-slate-300 px-3 py-2"
				>
					<option value="">-- Select --</option>
					<option>Cash</option>
					<option>Accounts Receivable</option>
					<option>Revenue</option>
					<option>Consulting</option>
					<option>Sales</option>
					<option>Rent Expense</option>
					<option>Utilities Expense</option>
					<option>Owner's Equity</option>
				</select>
			</div>

			<div class="md:col-span-2">
				<button
					type="submit"
					class="rounded bg-emerald-600 px-6 py-2 font-medium text-white hover:bg-emerald-700"
				>
					Save Transaction
				</button>
			</div>
		</form>
	</section>

	<section class="rounded-lg border border-slate-200 bg-white p-6 shadow-sm">
		<h2 class="mb-4 text-xl font-bold text-slate-800">Income Statement</h2>

		<div class="space-y-4">
			<div>
				<h3 class="mb-1 text-xs font-semibold tracking-wider text-slate-400 uppercase">Revenue</h3>
				{#each Object.entries(revenueAccounts) as [account, amount]}
					<div class="flex justify-between py-0.5 pl-2 text-sm text-slate-700">
						<span>{account}</span>
						<span>${amount.toFixed(2)}</span>
					</div>
				{/each}
				<div
					class="mt-1 flex justify-between border-t border-slate-100 pt-1 font-semibold text-emerald-700"
				>
					<span>Total Revenue</span>
					<span>${totalRevenue.toFixed(2)}</span>
				</div>
			</div>

			<div>
				<h3 class="mb-1 text-xs font-semibold tracking-wider text-slate-400 uppercase">Expenses</h3>
				{#each Object.entries(expenseAccounts) as [account, amount]}
					<div class="flex justify-between py-0.5 pl-2 text-sm text-slate-700">
						<span>{account}</span>
						<span>${amount.toFixed(2)}</span>
					</div>
				{/each}
				<div
					class="mt-1 flex justify-between border-t border-slate-100 pt-1 font-semibold text-rose-700"
				>
					<span>Total Expenses</span>
					<span>${totalExpenses.toFixed(2)}</span>
				</div>
			</div>

			<div class="flex justify-between border-t-2 border-slate-300 pt-2 text-lg font-bold">
				<span>Net Income</span>
				<span class={netIncome >= 0 ? 'text-emerald-700' : 'text-rose-700'}>
					${netIncome.toFixed(2)}
				</span>
			</div>
		</div>
	</section>

	<section class="rounded-lg border border-slate-200 bg-white p-6 shadow-sm">
		<div class="mb-4 flex flex-col justify-between gap-4 sm:flex-row sm:items-center">
			<h2 class="text-xl font-bold text-slate-800">Recent Transactions</h2>

			<div class="flex rounded-md shadow-sm">
				<button
					type="button"
					onclick={() => (activeFilter = 'all')}
					class="rounded-l-md border border-slate-300 px-4 py-2 text-sm font-medium {activeFilter ===
					'all'
						? 'bg-slate-200 text-slate-800'
						: 'bg-white text-slate-600 hover:bg-slate-50'}"
				>
					All
				</button>
				<button type="button" onclick={() => (activeFilter = 'page.server.js')} class="hidden"
					>Hidden Reference</button
				>
				<button
					type="button"
					onclick={() => (activeFilter = 'Revenue')}
					class="border-t border-r border-b border-slate-300 px-4 py-2 text-sm font-medium {activeFilter ===
					'Revenue'
						? 'bg-emerald-100 text-emerald-800'
						: 'bg-white text-slate-600 hover:bg-slate-50'}"
				>
					Income
				</button>
				<button type="button" onclick={() => (activeFilter = 'export const')} class="hidden"
					>Hidden Reference</button
				>
				<button
					type="button"
					onclick={() => (activeFilter = 'Expense')}
					class="rounded-r-md border-t border-r border-b border-slate-300 px-4 py-2 text-sm font-medium {activeFilter ===
					'Expense'
						? 'bg-rose-100 text-rose-800'
						: 'bg-white text-slate-600 hover:bg-slate-50'}"
				>
					Expenses
				</button>
			</div>
		</div>

		<div class="overflow-x-auto">
			<table class="w-full text-sm">
				<thead class="bg-slate-100 text-xs text-slate-600 uppercase">
					<tr>
						<th class="px-3 py-2 text-left">Date</th>
						<th class="px-3 py-2 text-left">Description</th>
						<th class="px-3 py-2 text-left">Debit</th>
						<th class="px-3 py-2 text-left">Credit</th>
						<th class="px-3 py-2 text-right">Amount</th>
						<th class="px-3 py-2 text-left">Type</th>
						<th class="px-3 py-2 text-right">Actions</th>
					</tr>
				</thead>
				<tbody>
					{#each filteredTransactions as t (t.id)}
						<tr class="border-t border-slate-200 hover:bg-slate-50">
							<td class="px-3 py-2">{t.date}</td>
							<td class="px-3 py-2">{t.description}</td>
							<td class="px-3 py-2">{t.debit}</td>
							<td class="px-3 py-2">{t.credit}</td>
							<td class="px-3 py-2 text-right">
								${t.amount ? Number(t.amount).toFixed(2) : '0.00'}
							</td>
							<td class="px-3 py-2">
								{#if classify(t) === 'Revenue'}
									<span class="font-medium text-emerald-700">Revenue</span>
								{:else if classify(t) === 'Expense'}
									<span class="font-medium text-rose-700">Expense</span>
								{:else}
									<span class="text-slate-400">Other</span>
								{/if}
							</td>
							<td class="px-3 py-2 text-right">
								<form method="POST" action="?/delete">
									<input type="hidden" name="id" value={t.id} />
									<button
										type="submit"
										class="rounded px-2 py-1 text-xs font-medium text-rose-600 hover:bg-rose-50"
										onclick={(e) => {
											if (!confirm('Are you sure you want to delete this transaction?'))
												e.preventDefault();
										}}
									>
										Delete
									</button>
								</form>
							</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	</section>
</div>
