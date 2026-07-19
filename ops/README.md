# Native Server Operations

This directory defines the small systemd workflow for the household AzerothCore
PlayerBots server. The services run as the existing `ditrain` owner and use the
private configuration already installed under `env/dist/etc/`.

The default policy is manual startup. Installing these units does not start the
servers and does not enable boot-time startup.

## Install or refresh the units

From `/mnt/data/wow-server/source`:

```bash
sudo install -o root -g root -m 0644 ops/systemd/azerothcore-auth.service ops/systemd/azerothcore-world.service ops/systemd/azerothcore.target /etc/systemd/system/
sudo systemctl daemon-reload
```

Installation copies only public service definitions. It does not read or copy
the private runtime configuration.

## Normal household workflow

Start both servers on demand:

```bash
sudo systemctl start azerothcore.target
```

The authserver is launched before worldserver. MySQL is a required dependency.

Check both server processes:

```bash
systemctl status --no-pager azerothcore-auth.service azerothcore-world.service
```

Follow their console output:

```bash
journalctl -f -u azerothcore-auth.service -u azerothcore-world.service
```

The application-specific logs remain in
`/mnt/data/wow-server/runtime/logs`.

Stop both servers gracefully:

```bash
sudo systemctl stop azerothcore.target
```

The ordering is reversed during shutdown: worldserver receives `SIGTERM` and
stops before authserver. The pinned core handles `SIGTERM` as a graceful
shutdown request. systemd allows up to five minutes for each process to drain
before treating it as stuck.

## Additional functions

Restart the complete pair:

```bash
sudo systemctl restart azerothcore.target
```

Restart only one failed or misbehaving component:

```bash
sudo systemctl restart azerothcore-auth.service
sudo systemctl restart azerothcore-world.service
```

Show recent logs without following them:

```bash
journalctl -n 200 --no-pager -u azerothcore-auth.service -u azerothcore-world.service
```

Both services use `Restart=on-failure` with a five-second delay. A crash is
retried automatically, while an intentional clean shutdown remains stopped.

Enable automatic startup at boot later:

```bash
sudo systemctl enable azerothcore.target
```

Return to manual startup without stopping a currently running game:

```bash
sudo systemctl disable azerothcore.target
```

Check the current boot policy:

```bash
systemctl is-enabled azerothcore.target
```

`disabled` is the intended initial result. Enabling or disabling boot startup
does not otherwise change the service definitions.

## Auction Bot Plus controls

Auction Bot Plus is configured through the ignored owner-only
`env/dist/etc/modules/mod_ahbot.conf`. Do not copy runtime configuration into
Git. Routine market behavior is automatic while worldserver is running.

The owner GM can use these supported in-game commands:

- `.ahbot update` advances one configured update tick; it does not necessarily
  run a seller or buyer action until that action's interval is reached.
- `.ahbot reload` reloads module configuration and rebuilds seller candidates.
- `.ahbot empty` removes bot-owned auctions, refunds current bidders, and
  cleans the generated item instances.

`.ahbot empty` does not cancel AHBot bids on player-owned auctions and cannot
reverse completed purchases. Treat those as normal auction-house transactions.

For an economy-only rollback, first run `.ahbot empty` while the module is
loaded, disable both seller and buyer, and stop the realm gracefully. Rebuild
and install without `mod-ah-bot-plus` only if the module itself must be
removed. The module adds no SQL schema, so ordinary rollback does not require a
schema migration. The verified pre-AHBot four-database backup is the
last-resort rollback because restoring it also discards every legitimate
post-backup account, character, mail, auction, and gameplay change.

These units intentionally do not add tmux or an attachable live console. Use
supported in-game administrative commands when appropriate. If an attachable
server console becomes a concrete need, add it as a separate reviewed step
rather than combining terminal-session management with the basic supervisor.
