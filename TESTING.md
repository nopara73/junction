# IO

Unittests must never actually mess with `~/.junction/...`. How to ensure this? Losing a wallet file due to running a unittest would be goddamn horrible ...

This gets into a broader question of how of "configuration". I need dev / test / staging / prod environments ...

I don't want to use flask config for this ...

I could make deletion less likely by adding a weird prefix like `python_unittest_wallet_blah_`

# Signer

I need a `Signer` class ...
- should this be able to associate itself with devices in HWI enumerate? Or should this be the client's job?

# Config

- flask needs it
- RPC() needs it (shouldn't, though)
    - tests should grab userpass (presumably `username:password`) from start_bitcoind and pass it into RPC()
    - RPC() shouldn't call `get_settings`
- Junction needs it

If `MultisigWallet` knows about the settings, then the settings file must be in place before 

Could have a `DataDir` or `Disk` class that knows how to save wallets (or anything with `.to_dict` and `.from_dict` methods. But this doesn't really fix the problem of `MultisigWallet.open` and `MultisigWallet.create` needing certain file structure in place. Maybe decorate these methods with `@require_datadir` methods? This decorator could create a data dir skeleton if datadir doesn't exist or lacks a settings file ...
- Well this decorator works for `MultisigWallet.create` but doesn't really work for `MultisigWallet.open`: what is there to open if no datadir exists??
- If just one method needs it, just make call `ensure_datadir()` directly ...

Should settings be per-wallet? Would someone ever want to use different nodes for different wallets? This sounds like premature optimization. One settings file is good.

The good thing about a DataDir class is that there is no ad-hoc logic anywhere else constructing file paths. The bad thing is that is sort of forces me into building a general-purpose serialization framework.

Should I just load everything into memory on startup? Ehh this would suck because then I wouldn't know what's the "source of truth" (perhaps)

# Testing

Maybe we can have a `JunctionTestCase` with a `setUpClass` that just asserts that `disk.DATADIR` isn't ~/.junction ?
