# Environment capture test hook

All 28 hooks documented by Git 2.50.1 share one capture script and capture
only `SECURITY_TEST_*` environment variables whenever Git invokes them.
Use synthetic values only: base64 is reversible, not encryption.
Requires Python 3.9+ and Git.

Enable for this checkout:

```sh
git config --local core.hooksPath "$(pwd)/.githooks"
```

Run without creating a commit:

```sh
SECURITY_TEST_SAMPLE=synthetic make capture-env
# Or invoke a specific hook:
SECURITY_TEST_SAMPLE=synthetic .githooks/pre-commit
```

The root Makefile invokes the shared payload in `.githooks/capture-env`.

Captures are base64-encoded JSON in the Git directory under
`security-test-env/capture-<hook>-*.b64` (directory mode 0700, file mode 0600).
Nothing is transmitted. No file is created when no test variables are set.
Delete capture files after testing.

Git has no universal hook: commands such as `log` and `diff` have none.
One action may invoke several hooks. Git's normal hook bypasses still apply.
This checkout is configured; clones require the enable command above.

Special hooks run only when their Git feature is configured:
`fsmonitor-watchman` requires `core.fsmonitor` pointing to its absolute path
and requests a full scan after capture; `proc-receive` requires
`receive.procReceiveRefs` and delegates reference updates back to Git;
`push-to-checkout` requires `receive.denyCurrentBranch=updateInstead` and runs
the documented `git read-tree -u -m HEAD <target>` checkout behavior.
Receive hooks execute in the receiving repository. Email and Perforce hooks
require `git send-email` and `git p4`, respectively.
