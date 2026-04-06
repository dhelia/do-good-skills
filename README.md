# do-good-skills

Open-source skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Built for founders and small teams who can't afford a legal department.

## What are Claude Code skills?

Skills are reusable prompt templates that teach Claude Code how to perform specific workflows. Think of them as runbooks that Claude follows — they turn multi-step processes into a single command.

## Available skills

| Skill | Command | Description |
|-------|---------|-------------|
| [WA Tax Filing](skills/wa-tax-filing/) | `/wa-tax-filing` | Prepare WA State sales tax filing from a Shopify CSV export. Computes B&O tax, sales tax by city, and flags anomalies. |
| [Tax Prep](skills/tax-prep/) | `/tax-prep` | Categorize business expenses from bank/credit card CSVs, generate QuickBooks .qbo import files, and produce a Schedule C-ready P&L. |
| [Schedule C](skills/schedule-c/) | `/schedule-c` | Generate IRS Schedule C with line-by-line totals and per-line-item transaction CSVs for TurboTax or paper filing. |
| [Audit Prep](skills/audit-prep/) | `/audit-prep` | Generate audit-proof tax documentation with organized binder, risk assessment, reconciliation checks, and 1099 review. |

## Installation

### Quick install (single skill)

Copy the skill folder into your Claude Code skills directory:

```bash
# Create the skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the skill you want
cp -r skills/wa-tax-filing ~/.claude/skills/
```

### Install all skills

```bash
# Clone the repo
git clone https://github.com/dhelia/do-good-skills.git

# Copy all skills
cp -r do-good-skills/skills/* ~/.claude/skills/
```

### Verify installation

Open Claude Code and type `/wa-tax-filing` — if the skill is installed correctly, it will load automatically.

## Usage

Each skill has its own `SKILL.md` with detailed instructions. The general pattern is:

```
/<skill-name> <input>
```

For example:

```
/wa-tax-filing ~/Downloads/q1_2026_sales.csv
```

## Configuration

Some skills support a `config.local.md` file for business-specific settings (entity name, tax nexus, etc.). These files are gitignored by default so your private configuration stays local. See each skill's `SKILL.md` for configuration details.

## Contributing

Contributions welcome! To add a new skill:

1. Create a folder under `skills/` with your skill name
2. Add a `SKILL.md` following the format of existing skills
3. Make sure it's general-purpose — scrub any personal or business-specific details
4. Update this README's skill table
5. Open a PR

## License

MIT

## Disclaimer

These skills assist with legal, tax, and business workflows but do not provide legal or tax advice. All output should be reviewed by qualified professionals before being relied upon for legal, tax, or financial decisions.
