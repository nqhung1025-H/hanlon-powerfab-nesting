# Hanlon PowerFab Nesting - development snapshot v1.12

This extension is still under development. This package is the working source
project for modifying, rebuilding and continuing the PowerFab/ABE integration.

The prebuilt development ZIP, including runtime binaries, is provided under
[Releases](https://github.com/nqhung1025-H/hanlon-powerfab-nesting/releases).
Clone this repository to edit and rebuild the source; download the release ZIP
when you also need the prepared PowerFab runtime files.

## Folder layout

| Folder | Contents |
| --- | --- |
| `src/ABE/` | ABE NC/SY2 imports, part matching and sheet-selection rules |
| `src/Configuration/` | Layout conditions and UI options |
| `src/Geometry/` | DXF/NC1 geometry, stock contours and remnant calculations |
| `src/Inventory/` | Inventory tracking, remnant lifecycle and inventory navigation |
| `src/Nesting/` | Plans, result models, material rules, names and batch calculations |
| `src/PowerFab/` | Extension entry point, PowerFab API, source data and purchasing |
| `src/Rendering/` | Nest and ABE preview image renderers |
| `src/UI/Forms/` | Main window and dialogs |
| `src/UI/Controls/` | Reusable preview controls |
| `src/UI/Layout/` | Screen, viewport and progress calculations |
| `src/UI/Theming/` | Hanlon styling |
| `Tests/Regression/` | Offline checks grouped by ABE, inventory, geometry, UI and saving |
| `Tests/Fixtures/` | Actual NC/SY2 files, XML and regression evidence |
| `Tests/Windows/` | Native Windows navigation-check source and project |
| `docs/` | Installation and result-format documentation |
| `docs/Validation/` | Current check output and prebuilt binary checksums |
| `lib/` | References required to compile the project |
| `bin/Release/net8.0-windows/` | Generated binaries; also included in the development ZIP release |
| `LICENSES/` | Dependency license |

The root keeps this README, `BUILD.cmd` and the main `.csproj` file. This new
repository starts from the v1.12 development snapshot. The existing `.gitignore`
keeps generated build output and local runtime logs out of commits. Production
namespaces and organised source locations are retained for continued development.

Extract the ZIP into a new empty folder, then use its project folder as your
working copy. This keeps the old flat source layout out of the organised folder.
The main project file and runtime output path are unchanged.

## Build and modify

Use Windows with the .NET 8 SDK, or Visual Studio with .NET desktop development.
The matching .NET 8-compatible PowerFab interface DLL is in
`lib/Tekla.PowerFab.Extension.dll`; Clipper2 is in `lib/Clipper2Lib.dll`.
For another PowerFab installation/version, use the matching interface reference.

Run `BUILD.cmd`, or run:

```powershell
dotnet build Hanlon.PowerFab.Nesting.csproj --configuration Release
```

The extension is built in `bin/Release/net8.0-windows/`. PowerFab supplies the
host interface at runtime; the interface reference in `lib/` is for compiling.
The SDK may report CA2255 for the intentional module initializer that resolves
the embedded Clipper2 library. The clean project builds successfully.

## Current behavior

ABE NC/SY2 import retains the source positions and rotations. The v1.12 snapshot
includes the supplied H0373 input fixes, one rectangular right/bottom remnant,
small holding-strip exclusion and the new 3000 x 1500 mm requisition-sheet
proposal when selected stock is too small. Save all nests remains a separate
user action. The existing saved-sheet navigation behavior is retained.

The ABE progress popup after sheet/remnant selection now shows only `Sheet N of
M` and `Importing: <sheet name>`, with Cancel available. The count uses the
current position in the imported batch, and the name comes from the ABE program.

ABE sheet and remnant names now exactly match the imported NC filename without
its extension. Capitalization, every space, separators and source suffixes are
preserved, for example `240175-02-PHASE90C-5MM`,
`240175-02-PHASE90C-12MM` and `H0413-NFF-PHASE1-TYPEB-10MM-01`.
Hanlon does not insert a space after PHASE, change letter case, normalize
separators, replace the thickness text or add/remove/renumber a source suffix.
The source name also takes precedence over older generated names when saving
an unsent draft. Material thickness mapping and backend IDs remain separate.
The same exact name is sent as the nest display name and future-remnant name.
Hanlon's preview summary and saved-sheet column use the label Sheet name.
The TrueShape Nest dialog in the supplied screenshot belongs to PowerFab:
its SaveNest API exposes the RemnantName value, but no UI-caption override.
That native caption remains Remnant Name. Existing saved PowerFab nests and
receipts are retained; the new format applies to newly prepared imports/saves.

The result action row has Clear nest immediately to the left of Import ABE
nest. The two buttons keep equal width and height as fonts/DPI change. Clear
nest clears the loaded review batch, sheet selector and preview images, then
focuses Import ABE nest. It leaves exported files and PowerFab saved records
available. It is inactive during reading, verification or API work.

The Select files / Select folder menu has subtle 4-DIP rounded corners. After
choosing files or a folder, NC/SY2 reading runs in the background with an inline
status and busy cursor; it does not open the initial generic progress popup.
The compact Sheet N of M / Importing: name dialog remains for verification
after sheet selection. Closing the window cancels pending background reading.

Sheet navigation now uses drawn chevrons centred inside their buttons; the
buttons align with the drawing viewport centre, excluding the tab strip.
ABE layouts use a cached vector preview for cutting paths and engraved plate
names. Zoom redraws the original strokes with screen-sized pens instead of
magnifying PNG pixels. This applies before sheet selection and when reviewing
saved/generated ABE results. The verified NC source is retained on result load
so switching back to a saved sheet also uses vector detail. Existing PNG files
remain available for PowerFab and other image viewers; remnant/non-ABE bitmap
previews retain their existing behavior.

ABE imports now use the imported NC cutting contours as the planning geometry.
PowerFab supplies the job/part identities, quantities, material and part sizes.
A different or unsupported PowerFab CNC/DXF drawing no longer blocks a valid
ABE layout. The original PowerFab CNC data and ABE source files are not edited.
Cut faces are reconstructed inside each SY2 placement; markings and lead-ins
do not become part outlines. Common cuts, straight/curved contours and unique
holding micro-joints up to 2 mm are supported. Missing or ambiguous outlines,
part-size/quantity mismatches, collisions and unsuitable stock remain errors.

The supplied 240175 Phase 90C sheets verify without any CNC API drawing fetch:
5 mm: 328 job parts and 18 packers; 6 mm: 43 job parts and 6 packers;
12 mm: 43 job parts and 26 packers. The old PowerFab 90_PL-2514 CNC outline is
240 x 210 mm; ABE and the PowerFab part record agree on 240 x 200 mm. The importer uses
the actual ABE outline and all four NC holes without resizing the old drawing.

ABE profile letters such as SPH, SPC and SS400 do not determine the PowerFab
grade or nominal thickness. NC and SY2 numeric profile settings must agree.
The program filename supplies the nominal thickness when present; otherwise
matching PowerFab parts resolve it using the following setup aliases:

| PowerFab thickness (mm) | ABE numeric setting (mm) |
| --- | --- |
| 1 | 1.2 |
| 2 | 2.0 |
| 3 | 3.2 |
| 4 | 4.5 |
| 5 | 4.5 |
| 6 | 6.0 |
| 8 | 8.0 |
| 10 | 9.0 |
| 12 | 12.0 |
| 16 | 16.0 |
| 20 | 19.0 |
| 25 | 25.0 |

Both 4 and 5 mm can therefore use SPH4.5. A 4MM/5MM filename distinguishes
them, or the matched PowerFab parts do so when no thickness is in the name.
If both nominal thicknesses match the same part without a filename, the
import remains ambiguous and requires selecting the intended PowerFab rows.
Saved stock, quantities and material retain the PowerFab nominal thickness
and grade; no source NC/SY2 settings are rewritten.

## Checks

Run the offline regression suite:

```powershell
dotnet run --project Tests/BatchSaveChecks.csproj --configuration Release
```

Run the native navigation checks on Windows:

```powershell
dotnet run --project Tests/Windows/PreviewNavigationChecks.csproj --configuration Release
```

The Windows check project rebuilds its executable from current source. It finds
this source root automatically and uses the checked-in XML fixture by default.
An alternative fixture XML path can be passed after `--`.

## Sheet-report process time

Import ABE nest reads PDF Sheet Reports in the selected folder and its subfolders.
Selecting NC/SY2 files also discovers PDF reports beside those files. No second
report-selection dialog is used. Each PDF page is read and matched using Data Name,
not the PDF filename, thickness alone, or page order. Separate PDFs and merged
PDFs are supported. Exact duplicate report times are accepted once; conflicting
report times remain unavailable for review. The reader does not guess a zero time.
For one combined report, keep that PDF anywhere in the selected nesting folder
or a subfolder; its filename does not need to match a sheet. Each report page
supplies only its matching sheet's runtime. A failed page is recorded with its
page number and reading stage while later pages continue.

Process time follows the packer count in the sheet summary, for example
`26 packers | Process time: 01:07:26`. It stays fixed above the drawing while
zooming and also appears in the Saved nests table. Values are per matched sheet report
and use HH:MM:SS, including durations longer than 24 hours. The supplied examples
are 02:38:02 (5 mm), 00:22:15 (6 mm), and 01:07:26 (12 mm). Nominal PowerFab
5 mm material remains separate from ABE's SPH4.5 profile.

The reports in the supplied folder contain bitmap sheet names and times. On
Windows 10/11, the extension uses the installed Windows PDF renderer and OCR
engine through Windows PowerShell. English OCR support must be available.
No cloud OCR service, external executable installation, or new PDF DLL is used.
A missing/unreadable report leaves the layout usable and labels its time as
unavailable. The PDF reader records problems in ImportedABE/sheet-report-reading.txt.
Version 1.12 fixes the PowerShell `AsTask` overload error during PDF rendering
and prevents async completion values from contaminating the reader's JSON output.
After replacing the runtime files and restarting PowerFab, re-import the ABE
folder to read the reports again; previously imported results are not backfilled.

Save all nests sends `Process time: HH:MM:SS` in PowerFab's `NestExtra2` field;
`NestExtra1` and the remnant name remain the exact source sheet name. SaveNest
exposes extra fields, not a dedicated machining-runtime property in the supplied
SDK. The same time is printed at the top of the nest PNG sent to PowerFab, so it
is visible when viewing the nest drawing. This is report metadata, not actual
machine time or a time-tracking record. Confirmed receipts retain the runtime.
The source PDF is archived beside the result with its SHA256 and report page.
Changed source reports are rejected on reload instead of silently changing time.
Existing saved PowerFab nests are not rewritten.

## Verification and remaining work

On 23 November 2026, all 930 offline regression checks passed, including 103
ABE cutting-geometry/profile checks, 55 name/mapping checks and 40 sheet-report checks. They replay all three supplied 240175 sheets,
earlier H0373 and H0306 layouts, curved/common cuts, small micro-joints,
source retention, 4/5 mm shared-profile resolution, saved-result reload and SaveNest XML generation. PowerFab
API calls and image bytes in the regression suite are mocked.
Another 14 PowerShell bridge checks passed using a portable async fixture and
the worker's actual overload-selection/await functions. They check action/result
overloads, waiting, error propagation and clean output. This does not execute
Windows WinRT PDF rendering or OCR.

The prebuilt production files are rebuilt from this v1.12 source. The production
build succeeds with the existing CA2255 advisory and zero errors. The separate
four SQLite grammar checks passed during the preceding validation; SQL is
unchanged. Native Windows popup/PDF/OCR execution and live PowerFab SaveNest/TFS were
not performed here. The PDF field matcher was exercised with captured OCR word
positions from the actual supplied PDFs; native Windows OCR still needs execution
on the PC. The native runner includes --reports-only for the separate PDFs,
merged PDF and a complete folder import with one merged report. Native remnant preview, live inventory use and workflow
acceptance require continued testing; this extension is still in development.
