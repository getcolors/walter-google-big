# walter-google-big

ARM Google Cloud Walter deployment in project `pocketcontext`, zone
`europe-west4-b`. Profile and dedicated R2 bucket are both `walter-google-big`.

- `c4a-standard-4-lssd`: 4 ARM cores, 16 GiB RAM, one 375 GiB Local SSD.
- 200 GiB Hyperdisk Balanced persistent root, Ubuntu 24.04 ARM64.
- Primary login `ubuntu`, plus `rose` and `jack`.
- Local SSD mounts at `/scratch`, with private per-user npm, uv, compiler-cache,
  temporary and build directories. Shell configuration selects those cache
  locations. Source, credentials, agent history and `/nix` stay persistent.

## Pinned deployment

The root `green` launcher is an exact copy of the installed
`.agents/skills/package-walter-green/green`; `skills-lock.json` records the
installation. Walter pins colors-compute transitively. No sibling working-tree
overrides are required.

Validate without credentials or cloud mutations:

```sh
./green build
./green create --dry-run
```

The private EU R2 bucket has been created, and the initial `./green create`
completed successfully on October 6, 2026. The Local SSD mount and enabled boot
service were verified, along with private scratch directories for all three
users. Emacs package bootstrap runs asynchronously.

Provider/backend credentials and `COLORS_PAR_WALTER_SSH_PASSPHRASE` are loaded
through the ignored `.envrc.private`. Its first line must be
`# -*- mode: sh; -*-`.
Never export `COLORS_PAR_PROFILE`.

Walter's compute pin uses the published `walter-empty-state-retry-20261006` release
lineage. It includes explicit C4A disk declarations and the failed-first-apply
retry fix, while retaining the SSH caller contract implemented by this package.
Newer compute main changes to
fresh SSH identity creation are not part of this dependency pin.

## Selected pricing option — not purchased

Three-year **resource-based** commitment, not Spot, in `europe-west4`:

| Resource | Quantity |
|---|---:|
| C4A vCPUs | 4 |
| RAM | 16 GiB |
| Titanium Local SSD | 375 GiB |

At the reviewed October 6, 2026 list prices, this is about $83.24/month for
the commitment plus $16.80/month for the 200 GiB root and $3.65/month for
IPv4: approximately **$103.69/month**, using 730 hours. Taxes, traffic and
R2 charges are extra. Without a purchased/active commitment, the equivalent
on-demand total is approximately **$205.36/month**. Merely setting the machine
type does not enable a discount.

The commitment cannot be cancelled after purchase and remains payable even
without a VM. The deployment does not create, renew, or delete commitments.
The VM and bucket are deployed. No reservation or commitment was purchased;
the VM currently uses on-demand billing.

For a separately authorized future purchase, after verifying quota, existing
commitments and actual pricing, the intended CLI request is:

```sh
gcloud compute commitments create walter-google-big-c4a-36m \
  --project=pocketcontext \
  --region=europe-west4 \
  --type=general-purpose-c4a \
  --plan=36-month \
  --resources=vcpu=4,memory=16GB,local-ssd=375GB \
  --no-auto-renew
```

Sources: [Google VM pricing](https://cloud.google.com/products/compute/pricing/general-purpose),
[disk pricing](https://cloud.google.com/compute/disks-image-pricing),
[IPv4 pricing](https://cloud.google.com/vpc/network-pricing),
[commitment purchase](https://docs.cloud.google.com/compute/docs/committed-use-discounts/purchase-commitments),
[Local SSD persistence](https://docs.cloud.google.com/compute/docs/disks/local-ssd).
