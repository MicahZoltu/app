<script lang="ts">
  import { createEventDispatcher, onMount } from 'svelte';
  import type { StepComponentEvents } from '$lib/components/stepper/types';
  import type { Writable } from 'svelte/store';
  import type { State } from '../../claim-project-flow';
  import transact, { makeTransactPayload } from '$lib/components/stepper/utils/transact';
  import GitProjectService from '$lib/utils/project/GitProjectService';
  import { getCallerClient } from '$lib/utils/get-drips-clients';

  const dispatch = createEventDispatcher<StepComponentEvents>();

  export let context: Writable<State>;

  onMount(() =>
    transact(
      dispatch,
      makeTransactPayload({
        before: async () => {
          const gitProjectService = await GitProjectService.new();

          const setSplitsAndEmitMetadataBatch = await gitProjectService.buildBatchTx($context);

          return { callerClient: await getCallerClient(), setSplitsAndEmitMetadataBatch };
        },
        messages: {
          duringBefore: {
            message: 'Preparing to claim project...',
          },
        },
        transactions: ({ callerClient, setSplitsAndEmitMetadataBatch }) => ({
          transaction: () => callerClient.callBatched(setSplitsAndEmitMetadataBatch),
          waitingSignatureMessage: {
            message: 'Waiting for you to confirm the transaction in your wallet…',
            subtitle:
              "This transaction applies your project's splits, sets metadata, and concludes the claiming process.",
          },
        }),
      }),
    ),
  );
</script>
