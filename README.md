# JSONGenerator: Multi-Station PagerDuty Demo Configuration

A Perl-based configuration generator that transforms CSV definitions and JSON event payloads into JavaScript data structures for a multi-station PagerDuty demonstration environment.

Developed by **Eric Burns** to extend [Martin Stone's PagerDuty AWS Demo](https://github.com/martindstone/awsdemo) for simultaneous demonstrations by multiple Sales Engineers and Account Executives.

## The Problem

The original demonstration application allowed presenters to trigger simulated monitoring alerts and predefined sequences of incidents in PagerDuty.

For a larger event, we needed multiple demonstration stations operating independently. Each presenter needed to walk through their own incident management scenarios without interfering with another presenter's demonstration.

The configuration involved nested JavaScript objects representing stations, integrations, routing keys, event payloads, and timed sequences.

Manually duplicating and modifying this configuration for every station would have required maintaining approximately 1,500 lines of repetitive, multidimensional data.

That approach was time-consuming, difficult to validate, and prone to errors such as missing commas, incorrect routing keys, and inconsistent configuration between stations.

## The Solution

Rather than manually maintain the generated JavaScript, I built a Perl utility that uses small, human-editable configuration files to construct the data structures.

The generator separates:

- **Stations (personas):** Individual presenters or demonstration environments.
- **Services:** Monitoring integrations such as CloudWatch, CloudTrail, GuardDuty, and New Relic.
- **Routing keys:** Integration-specific identifiers associated with each station.
- **Payloads:** Reusable JSON representations of monitoring alerts.
- **Demonstrations:** Named incident scenarios.
- **Flows:** Ordered events with configurable delays between alerts.

The script reads the configuration files, resolves their relationships, and generates the JavaScript structures consumed by the demonstration application.

This substantially reduces the manual effort required to add stations, alerts, or scenarios while keeping the underlying demonstration application unchanged.

## Real-World Result

The approach proved valuable during the event.

When another demonstration station was needed at the last minute, adding its configuration and regenerating the JavaScript took seconds instead of requiring extensive manual duplication.

The same event scenarios and alert payloads could be reused across stations while routing incidents to the appropriate PagerDuty integrations.

This was a practical example of investing in automation to reduce operational risk, improve consistency, and support last-minute changes.

## How It Works

The primary script, `createElegant.pl`, reads configuration and payload files from two directories:

| File | Purpose |
|---|---|
| `csv/personas.csv` | Defines the demonstration stations and their identifiers |
| `csv/services.csv` | Defines the available integration types |
| `csv/persona2integration2key.csv` | Associates each station with its integration routing keys |
| `csv/payloads.csv` | Maps human-readable alert names to JSON payload filenames |
| `csv/demos.csv` | Defines the available demonstration scenarios |
| `csv/flows.csv` | Defines ordered events, integrations, payload references, and delays |
| `JSON/*.json` | Individual sample alert payloads |
| `createElegant.pl` | Perl configuration generator |

The generator produces three files:

| Output | Purpose |
|---|---|
| `payloads.js` | JavaScript objects containing the example alert payloads |
| `services.js` | Station-to-service mappings and integration routing keys |
| `sequences.js` | Nested structures defining timed incident sequences for each station |

These are JavaScript files containing objects and references, not standalone JSON documents.

The outputs are intended to be used with the browser-based [awsdemo application](https://github.com/ericburnsonline/awsdemo).

## Running the Generator

### Requirements

- Perl
- The configuration files in the `csv/` directory
- The corresponding alert payload files in the `JSON/` directory

The existing script uses Perl's built-in file handling and string-processing capabilities without requiring a separate CSV parsing library.

From the repository root, run:

`perl createElegant.pl`

The script reads its input files and writes `payloads.js`, `services.js`, and `sequences.js` into the current directory, replacing any existing files with those names.

Console output reports the number of stations, services, and payloads processed, along with completion messages for the generated files.

The generated files can then be placed alongside `index.html` and `awsdemo.js` in the demonstration application.

The original browser application was developed by Martin Stone and is maintained separately.

## Adding or Modifying Configuration

### Add a demonstration station

1. Add an entry to `csv/personas.csv` containing a unique numeric identifier and display name.
2. Add the station's integration mappings to `csv/persona2integration2key.csv`.
3. Run `createElegant.pl` to regenerate the JavaScript files.

For example, a station entry follows this format:

`8,08 Demo (Example)`

The integration mapping file associates that station identifier with each required service and its routing key.

The generator creates the station-specific structures and expands the predefined incident sequences for the additional presenter.

### Add an alert payload

Create a JSON payload file in the `JSON/` directory using a short filename without spaces, such as:

`ssh_brute_force.json`

Then add its display name and filename identifier to `csv/payloads.csv`:

`"GuardDuty: SSH Brute Force Attack", ssh_brute_force`

Run the generator to update `payloads.js`.

The new event becomes available for selection in the application's Payload dropdown.

To include it in a predefined demonstration sequence, also add the appropriate entry to `csv/flows.csv`.

### Add a demonstration scenario

Add a unique scenario key and descriptive title to `csv/demos.csv`:

`GD5SF,"GD 5 Sample Findings"`

Then add the sequence of events to `csv/flows.csv`.

Each row contains:

`Demo Key, Service Key, Payload Short Name, Delay`

For example:

`GD5SF,GD,ssh_brute_force,0`

The order of entries matters. Events belonging to the same flow should remain grouped together in the CSV file.

The delay value is expressed in seconds.

The generator uses these definitions to create station-specific sequences in `sequences.js`.

### Add a service or integration

Add a short identifier and display name to `csv/services.csv`:

`GD,GuardDuty`

Then add the corresponding routing-key mappings for each station that uses the service in `csv/persona2integration2key.csv`.

The service identifier is referenced by demonstration flows and integration mappings.

Regenerate the JavaScript to apply the configuration changes.

## Implementation Notes and Limitations

This was developed as a practical utility for a specific demonstration environment rather than a general-purpose configuration framework.

Some implementation characteristics are worth noting:

- **Perl-based generation:** Uses simple loops, hashes, arrays, and file operations to assemble nested JavaScript objects.
- **CSV parsing:** Uses basic string splitting rather than a dedicated CSV library. Input formatting must follow the conventions expected by the script.
- **Shared scenarios:** Flow definitions are reused for each configured station.
- **Station-specific routing:** Integration keys are resolved independently for each station.
- **Basic validation:** The script reports certain missing payload definitions and malformed integration mappings, but does not comprehensively validate all input.
- **Browser compatibility:** The generated JavaScript reflects the assumptions of the original demonstration application and has not been independently validated against modern browsers.

No automated test suite is included.

## Security and Configuration

This repository contains historical PagerDuty integration routing-key configuration.

Before adapting or reusing this project, replace the original routing keys with credentials belonging to a dedicated demonstration environment.

For a modern deployment, credentials should be managed separately from source control, using environment variables or appropriately protected configuration files.

Because the original demonstration application runs in a browser, any routing keys included in client-side JavaScript are accessible to users of that application. Sensitive operations requiring confidential credentials should instead be handled through a server-side component.

## Project Status

**Complete / Historical**

This repository documents the configuration generator developed for the original multi-station PagerDuty demonstration environment.

Its value is primarily as an example of practical automation, configuration-driven development, data transformation, and solving a real operational problem under event constraints.

It is retained as a historical technical project rather than a currently maintained, production-ready utility.

## Related Projects and Attribution

**Original demonstration application:** Martin Stone

[Martin Stone's awsdemo](https://github.com/martindstone/awsdemo)

**Fork of the demonstration application:**

[Eric Burns' awsdemo](https://github.com/ericburnsonline/awsdemo)

**Multi-station configuration generator:** Eric Burns

[JSONGenerator](https://github.com/ericburnsonline/JSONGenerator)

The original application provided the browser interface and incident-triggering behavior. JSONGenerator was developed to automate the station-specific configuration and event sequences required to operate the demonstration across multiple presenters.
