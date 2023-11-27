![tx](https://github.com/Sovereign-Labs/docs-draft/assets/8730839/413b5e80-9dfd-4771-bcda-b5b4e735f4e9)

```dot
digraph tx_flow {
    graph [
        label = "Sovereign SDK Wallet Snap Transaction Workflow"
        labelloc = t
        fontname = "Helvetica,Arial,sans-serif"
        layout = dot
        rankdir = LR
        newrank = true
    ]
    node [
        style=filled
        shape=rect
        fontname="Helvetica,Arial,sans-serif"
    ]
    edge [
        arrowsize=0.5
        fontname="Helvetica,Arial,sans-serif"
    ]

    subgraph cluster_setup {
        sov_snap_generator [label="sov-snap-generator"]
        wallet_signer [label="Signer"]

        label = "Setup"
    }

    subgraph cluster_start {
        style=filled;
        color=lightblue;

        ux [label="UX"]

        label = "Entrypoint"
    }

    subgraph cluster_sov_sequencer {
        sequencer_batch_builder [label="BatchBuilder"]
        sequencer_client [label="Client"]

        sequencer_batch_builder -> sequencer_client [label="dry_run"]

        label = "sov-sequencer"
    }

    subgraph cluster_sov_rollup_interface {
        rollup_state_transition_function [label="StateTransitionFunction"]
        rollup_da_service [label="DAService"]

        label = "sov-rollup-interface"
    }

    subgraph cluster_sov_stf_runner {
        runner [label="Runner"]
        runner_prover [label="ProverService"]
        runner_finalized_block [label="Finalized Block"]

        label = "sov-stf-runner"
    }

    subgraph cluster_sov_db {
        db_slot_commit [label="SlotCommit"]

        label = "sov-db"
    }

    subgraph cluster_sov_modules_api {
        module_apply_blob_hooks [label="ApplyBlobHooks"]
        module_tx_hooks [label="TxBlobHooks"]

        label = "sov-modules-api"
    }

    subgraph cluster_end {
        style=filled;
        color=lightgreen;

        da_service [label="DA layer implementation"]
        storage [label="Storage"]
        module_runtime [label="RuntimeCall"]
        module_hooks [label="Hooks"]

        label = "Endpoint"
    }

    ux -> wallet_signer [label="JSON(RuntimeCall)"]
    wallet_signer -> ux [label="borsh(tx)"]
    ux -> sequencer_batch_builder [label="RPC(sequencer_acceptTx(borsh(tx)))"]
    ux -> sequencer_client [label="RPC(sequencer_publishBatch)"]
    sov_snap_generator -> wallet_signer
    sequencer_client -> rollup_da_service [label="send_transaction"]
    rollup_da_service -> runner

    runner -> rollup_state_transition_function [label="apply_slot/apply_blob"]

    rollup_state_transition_function -> module_apply_blob_hooks [dir="both"]
    rollup_state_transition_function -> module_tx_hooks [dir="both"]
    rollup_state_transition_function -> runner_finalized_block [label="[TransactionReceipt{events, receipt}]"]
    rollup_state_transition_function -> module_runtime [label="dispatch_call", dir="both"]

    module_apply_blob_hooks -> module_hooks [dir="both"]
    module_tx_hooks -> module_hooks [dir="both"]
    runner_finalized_block -> runner_prover
    runner_finalized_block -> db_slot_commit
    db_slot_commit -> storage
    runner_prover -> da_service
}
```
