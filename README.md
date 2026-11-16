# Wryd Company Dagu Workflows

This is the `main` branch of the `workflows` repo. It contains:

- re-useable sub-dags
- re-useable scripts
- a [toha]() template for initializing new workspace branches
- sub-dag tests
## Workspace updates

Every workspace keeps itself current with its shared repositories. Its Trowel
(configured by workspace-ansible's `templates/etc/trowel/config.yml.j2`) takes
each push GitHub reports for them and enqueues a DAG in `sub-dags/`:

| Push to | DAG | Runs |
| --- | --- | --- |
| `workspace-ansible` | `workspace-update-ansible` | `sudo ansible-sync`: pulls the playbook and reapplies it |
| `workspace-dotfiles`, `skills` | `workspace-update-dotfiles` | `chezmoi update --init`: pulls the dotfiles and applies them; every apply fast-forwards the skills clone |
| `workflows` | `workspace-update-workflows` | `git pull --ff-only` in `/workflows/main` |

Trowel passes the push's `REF` and `SHA` as params. Pushes carry no branch in
their NATS subject, so each DAG's step skips any `REF` but `refs/heads/main`:
the run succeeds with the step skipped. The DAGs share the `workspace-update`
queue, so they never run at once.

`workspace-update-dotfiles` and `workspace-update-workflows` fail rather than
update when their checkout is not a clean `main`, so local work is never
rebased or merged.

Run one by hand from the Dagu UI, or:

```sh
dagu enqueue --config /etc/dagu/config.yml /workflows/main/sub-dags/workspace-update-dotfiles.yml
```

Test: `tests/workspace-update/preconditions.sh` dry-runs the DAGs and checks
which refs run the step.
