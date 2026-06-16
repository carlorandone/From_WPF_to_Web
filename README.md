# From WPF to the Web — Companion Reference Samples

This repository is the **companion to the Medium article**:

> **"From WPF to the Web: Reusing Your MVVM Investment in Blazor Server and React SPAs"**  
> *A pragmatic, code-first migration playbook for .NET teams modernizing legacy desktop apps*  
> by Carlo Randone, IBM Consulting Italy — 2026

📰 **Read the article on Medium**: [link to be added once published]

The repository contains **three reference samples** — packaged as standalone ZIP archives — that implement the same business use case (an inbound documents form) across three different presentation stacks. Each sample is designed to be read side by side with the corresponding sections of the article.

## The three samples

### 1. `InboundDocs-Wpf.zip` — WPF Baseline

A traditional **.NET 8 WPF** application with classic MVVM:

- XAML views with two-way data binding
- `ViewModelBase` with `INotifyPropertyChanged`
- `RelayCommand` exposing `ICommand` to the View
- `WpfDialogService` (`MessageBox`-based)
- xUnit + Moq + FluentAssertions test suite

This is the **starting point**: the form as it would have been built in a classic WPF MVVM codebase before any web migration was considered.

### 2. `InboundDocs-BlazorServer.zip` — Blazor Server Migration

An **ASP.NET Core 8 Blazor Server** application that reuses the same `InboundDocs.Core` library from sample #1, with no changes to the Models, Services or ViewModels. Demonstrates:

- ViewModel registered as `Scoped` (per-circuit) service
- Explicit `INotifyPropertyChanged` subscription in `OnInitialized` / unsubscription in `Dispose`
- `CascadingValue` pattern to share the ViewModel across child components
- `BlazorDialogService` with `TaskCompletionSource` and a globally placed `DialogHost.razor`
- `InvokeAsync(StateHasChanged)` defensive routing for thread-safety

Corresponds to **Part I (Chapter 4)** of the article.

### 3. `InboundDocs-ReactSpa.zip` — React SPA + .NET API Migration

A **React + Vite + TypeScript** Single-Page Application backed by an **ASP.NET Core 8 Minimal API**. The same `InboundDocs.Core` library still lives on the server side (consumed by the API), while the ViewModel is reimplemented in TypeScript as a **custom React hook**. Stack:

- **Zustand** for client-side state
- **TanStack Query** for server state and mutations
- **React Hook Form** + **Zod** for forms and validation
- **Radix UI** for accessible dialog primitives
- **React Router** for navigation
- API tier with DTOs, CORS, and Swagger UI

Corresponds to **Part II (Chapter 5)** of the article.

## Central property — verified empirically

The **`InboundDocs.Core` class library is byte-for-byte identical** across all three samples. This was verified with a SHA256 hash check across the 12 source files of `Core/` and `Core.Tests/`, which returned a 12-out-of-12 match:

| File | SHA256 (first 12 chars) |
|---|---|
| `InboundDocs.Core.csproj` | `4837714AECA1` |
| `Models/InboundDocument.cs` | `4C1060BA8370` |
| `Models/InboundDocumentLine.cs` | `5D46FA3CA0D7` |
| `Services/IDialogService.cs` | `2BDEDC2D6BDD` |
| `Services/IInboundDocumentRepository.cs` | `918788B5B04D` |
| `Services/InboundDocumentValidator.cs` | `1E51B11B561F` |
| `Services/InMemoryInboundDocumentRepository.cs` | `A220FE63A39F` |
| `ViewModels/ViewModelBase.cs` | `1A53A0970530` |
| `ViewModels/RelayCommand.cs` | `4E864C1FCC6C` |
| `ViewModels/InboundDocumentViewModel.cs` | `DA5236269D4B` |
| `InboundDocs.Core.Tests.csproj` | `80670F90F1F9` |
| `InboundDocumentViewModelTests.cs` | `8A535F1A8D7B` |

The same Models, the same Services, the same ViewModels, the same xUnit test suite — three hosts, one shared core. This is the article's central thesis made concrete and verifiable.

## How to use this repository

1. **Download** the ZIP for the sample that matches the article section you are reading.
2. **Extract** it to a clean folder (ideally a path that does **not** contain special characters like `$`; we recommend `C:\dev\` or `~/dev/` for cross-platform safety).
3. **Open the `README.md` inside each ZIP** for run instructions, architectural notes, and the project-specific "Fix log".

Each sample is self-contained: its own `InboundDocs.sln`, its own copy of `InboundDocs.Core/`, and its own `InboundDocs.Core.Tests/`. You can build and run any sample independently.

## Prerequisites

- **.NET 8 SDK** — required by all three samples
- **Windows 10/11** — required only by sample #1 (WPF)
- **Node.js 20+** — required only by sample #3 (React SPA)
- Modern browser — recommended **Microsoft Edge** or **Chromium-based**

## Quick start

### Sample 1 — WPF (Windows only)

```powershell
Expand-Archive .\InboundDocs-Wpf.zip -DestinationPath .
cd InboundDocs-Wpf
dotnet run --project InboundDocs.Wpf
```

### Sample 2 — Blazor Server

```powershell
Expand-Archive .\InboundDocs-BlazorServer.zip -DestinationPath .
cd InboundDocs-BlazorServer
dotnet run --project InboundDocs.BlazorServer
```

Then open `http://localhost:5000/inbound-document/R-12345` in your browser.

### Sample 3 — React SPA + .NET API (requires two terminals)

**Terminal 1 — API:**

```powershell
Expand-Archive .\InboundDocs-ReactSpa.zip -DestinationPath .
cd InboundDocs-ReactSpa
dotnet run --project InboundDocs.Api
```

**Terminal 2 — React SPA:**

```powershell
cd InboundDocs-ReactSpa\inbound-docs-spa
npm install
npm run build
npm run preview
```

Then open `http://localhost:4173/inbound-document/R-12345` in your browser.

## Repository structure

```
.
├── README.md                            ← this file
├── InboundDocs-Wpf.zip                  ← sample 1 (Project 1 of the article)
├── InboundDocs-BlazorServer.zip         ← sample 2 (Project 2 of the article)
└── InboundDocs-ReactSpa.zip             ← sample 3 (Project 3 of the article)
```

Each ZIP, once extracted, contains a `README.md` with full run instructions, smoke-test scenarios, and a project-specific "Fix log" documenting the subtle bugs we hit while integrating the sample.

## Smoke test — what to look for after a successful run

Once you have any sample running and your browser open on `/inbound-document/R-12345`, the following should happen:

1. The form preloads with **Receipt code** `R-12345`, **Supplier code** `SUP-001`, and **2 lines** (`ITM-100` and `ITM-200`), totaling **$1,250.00**.
2. Click **Add line** → a new row appears with default values.
3. Edit the quantity or unit price of the new line → the **Total amount** recalculates immediately.
4. Click **Save** → a modal dialog asks *"A document with this receipt code already exists. Overwrite?"*.
5. Click **Yes** → a second dialog confirms *"The document was saved successfully."*.
6. Click **OK** → the form returns to its idle state.
7. (Sample 2 and 3 only) Refresh the page → the new line persists.

If you can complete this smoke test on all three samples, you have a full end-to-end validation of the article's central thesis.

## License

All sample code in this repository is provided as-is for educational purposes accompanying the article. You are free to use, modify, and adapt it for your own learning and projects. See the `LICENSE` file (if present) for the formal license terms; if no `LICENSE` file is present, treat the contents as under the **MIT License**.

## Feedback and discussion

If you have questions, corrections, or your own war stories from a WPF-to-web migration, please open an Issue on this repository or join the discussion under the Medium article. We genuinely want to hear how this plays out in other organizations and other migrations.

— Carlo Randone, IBM Consulting Italy
