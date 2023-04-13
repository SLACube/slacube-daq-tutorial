# Tutorial
## Working Enironment

Although **slacube** script works in any folders, a custom working directory
(set to `$SLACUBE_WORKDIR`) is required to store run configurations, temporary
data files and diagonstic outputs.

To create an working environment:
```
   $ slacube env create some_name
```

The environment becomes active **ONLY** after sourcing the setup script:
```
   $ source $(slacube env curr)
```

**-- INSERT SCREENCAST HERE --**

Notes:
- The same working environment is "reusable", provided that the running
  condition remains unchanged.
- Remember the name of the directory (make something unique to you).
- `slacube env curr` shows the setup script for last modified directory, which
  is the most likely environment for the next operation.
- `slacube env list` shows a list of avaliable environments. Source any of them
  to activate.

See `slacube env help` for details.

## Interlude: How to Read Help Pages
**-- INSERT SCREENCAST HERE --**

## Controller Configuration

A tile controller configuration defines the readout topology (i.e. Hyrda
network) of the LArPix ASICs. Once the tile config is generated, it can be used
for the entire run unless there is a re-routing of the network to avoid
problematic chips.

To create a new hydra network:
```
   $ slacube hydra create
```
It will generates a json file, somehting like `tile-id_1-autoconfig.json`.

**-- INSERT SCREENCAST HERE --**

Keep an eye on the output. You should see **FOUR** good root chips:
```
... ...
good root chips:  [11, 41, 71, 101] 
... ...
```

Sometimes one of the root chip does not work, espeically right after power
cycle. In such case, you may abort the job and try it again. The LArPix would
probably works without all 4 root chips, but missing one could indicates some
issues on the tile.

Set the tile config for later operations:
```
   $ slacube hydra set
```
Choose the desired config file and press "Enter" to confirm.
**-- INSERT SCREENCAST HERE --**

Plot the hydra network
```
   $ slacube hydra plot
   Save figure to hydra_6437c4ec.png
```

![Hydra Network](figures/hydra_6437c4ec.png)

Notes:
- X-forwarding over SSH maybe slow. Try `sshfs` or other `sftp` clients to view
  the image.
- Only four root chips 11,41,71,101 are connected externally for readout.
- The numbers in the upper corners of the root chips are `io_channel`, which
  gives the communication path for each chip. 
- Each chip belongs to one `io_channel` as indicated in different
  colors.
- If a chip is greyed out, it is not "reachable" from outside. It should not
  be happened in normal operation.
- A **chip key** is an unique identifier in form of `{io_group}-{io_channel}-{chip}`.
  - `io_group=1` for the SLACube setup.
  - `io_channel` is indicated in the figure.
  - `chip` ranges from 11 to 110 (a total of 10x10 ASICs per tile)
- For example, here are valid chip keys: 1-1-13, 1-2-55, 1-3-86, 1-4-101.
- Chip keys for the same ASIC may vary if a different controller config is used.

See `slacube hydra help` for details.

## Trigger Rate Test
 
Trigger rate test identified anomalous channels due to high leakage current. It
produces a bad channel list, where problematic channels are masked in further
operations.

To start a trigger rate test:
```
   $ slacube rate-test start
```
**-- INSERT SCREENCAST HERE --**

A few data files `trigger_rate_*.h5` and a json file started with
`trigger-rate-DO-NOT-ENABLE-channel-list-...` are produced.

To update the new bad channel list:
```
   $ slacube blacklist set
```
**-- INSERT SCREENCAST HERE --**

Note:
- Use `slacbue cfg` to check the current run configurations, including
  environmental variables and runtime files/directories.

See `slacube rate-test help`, `slacube blacklist help` and `slacube cfg help`.

## Pedestal Test

Pedestal test identified channels with large pedestals. A new blacklist is
generated after the test.

Start a quality control (QC) test:
```
   $ slacube pedestal start-qc
```
A short pedestal run is taken `pedestal_...h5`.
A new bad channel list is generated `pedestal-bad-channels-...json`.
Another short pedetal run is taken with the updated bad channe list `recursive_pedestal_...json`.

Make pedestal plots:
```
   $ slacube pedestal plot
```
Select pedestal file, and press "Enter".

**-- INSERT FIGURES HERE --**

Notes:
- For further diagonstic, a csv file is provided from the plot command.
- Do not pay attention to the empty patterns on the pedestal figures. These are
  known features.
- The QC test only mask the bad channel with large pedestal > 125 ADC. It is
  possible to mask channels with high std, `start-qc --apply_noise_cut --noise_cut_value MAX_STD`,
  which is less often used.
- You can mask individual channel manually using the chip-key + channel (see later).

Set the latest bad channel list:
```
   $ slacube blacklist set
```
Pick the latest `pedestal-bad-channels-...json` and press "Enter".

Do a second check with another pedeatal run (without QC test):
```
  $ slacube pedestal start
```
It only take a single pedestal run and does not produce a new bad channel list.
Plot the latest pedestal file and check the figure.

At this point, most of the bad channels are masked automatically by rate and
pedestal QC tests. A few more channels might still need be added manually. As
for safety break point, it is recommended to duplicate the bad channel list at
this stage.

Copy the current bad channel list:
```
   $ slacube blacklist copy
```
A new copy of the json file is set for later operation.

See `slacube pedestal help`.

## Interlude: How to Add Bad Channel

![hydra_6437c4ec](https://user-images.githubusercontent.com/55715/231730721-b2c89b6e-bbed-44f5-b91a-7d861f98acad.png)
