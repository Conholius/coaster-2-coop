# Runtime progression: diagnostics 007–034

**Consolidated 2026-10-21.** [Status](STATUS.md) · [Transport](../networking/TRANSPORT.md)

## Provenance and limits

The owner ran the diagnostics manually in a copied PC2 test installation and supplied private logs for review. This is manually initiated in-game diagnostic coverage, not a two-client automated co-op QA suite. Earlier 005/006 runs were explicitly outside Sandboxie. Do not infer later per-run process isolation from copied paths; verify the actual launch context separately.

Environment reported in project logs: Steam / Windows, PC2 1.11.4.1011357, ACSE 0.729, ForgeUtils 2.7.0, ACSEDebug 0.014. Executable and packaged-mod hashes are not established here. Raw logs and game files stay private.

007–029 findings consolidate the prior log reviews recorded in the research conversation. During this update the retained 022/025 persistence logs, both 030 logs, and current 034 log were also rechecked. Missing older standalone logs are not represented as newly rerun tests.

## Diagnostic ledger

| Diagnostic | Action / result | Boundary |
| --- | --- | --- |
| 007 | CreateUndoOperationsHierarchy invoked successfully in park | Beyond mere API presence |
| 008 | GetFullUndoChangeList invoked on the returned hierarchy | No placement yet |
| 009 | CreateNewPart for AQ_Rock_04; entity-request completion awaited | No preview/commit |
| 010 | StartEditingPart_Scenery(createObject) succeeded; transform methods inspected | No position change |
| 011 | GetTransform/GetScale/GetQuaternion and transform:GetPos read | vector components require getters |
| 012 | vector3 getters, transform snapshots, controlled SetTransform/readback/restoration | Observed behavior, not universal deep-copy proof |
| 013 | Movement survived another Advance; original transform restored | No preview/commit |
| 014 | Preview returned two tokens; completion awaited; cancellation completed | No commit |
| 015 | CanCommitPreview returned true; preview cancelled | Invalid-case failure reasons not exercised |
| 016 | One CommitPreview, completion, idle and undo checkpoint | Checkpoint is not park save |
| 017 | Part IDs captured; world entity mappings/positions checked over two Advances | Entity IDs remain local/transient |
| 018 | AQ_Rock_04 committed at (10, 5, 10), stable over two Advances | No save |
| 019 | New local save; manual reload with verify-only code found part 1182 and resource at (10, 5, 10) | No new part created during verification |
| 020 | Existing 1182 edited to (15, 5, 10), held across Advance and restored to (10, 5, 10) | No preview/commit/save |
| 021 | Existing part moved and committed once at (15, 5, 10); checked twice | No save |
| 022 | Same move committed once, saved as a new local park; manual reload verified resource/position over two Advances | Persistence of this tested part |
| 023 | Destroy prepared deletion; preview/cancel completed; original part restored at (15, 5, 10) | Entity mapping can be recreated with a different entity ID |
| 024 | Deletion committed once; part and entity mappings absent over two Advances | No save |
| 025 | Deletion committed once, saved as new local park; manual reload verified continued absence twice | Durable deletion in this save |
| 026 | Two equal-resource rocks assigned own keys and distinct part IDs; independent move/delete previews and cancellation/restoration | Preview isolation test |
| 027 | Create A, create B, move A, delete B: each action committed once; other part checked unchanged | Four commits, no save |
| 028 | Eight inputs; four equal-payload duplicates skipped; four actions committed | In-memory session deduplication |
| 029 | Four ordering errors and four ID conflicts rejected; four valid commits; A at (15, 5, 10), B absent | No durable journal or crash recovery |
| 030 | Same twelve-command scenario in A/B-labelled separate runs; twelve matching canonical snapshots (120 STATE records each) | Not simultaneous clients or transport |
| 031 | Standard io/os and LuaSocket unavailable; package.loadlib function present but not called | File/socket tests skipped, not passed |
| 032 | Raw namespace enumeration incomplete; protected metatables | No network invocation |
| 033 | Five candidate fields resolve as function with normal field access inside pcall | No invocation/signature validation |
| 034 | Five controls function, nine invented names nil, no errors; Completed=true | Tested resolver distinguishes these names |

## Confirmed scenery lifecycle

This is an independently written call-sequence summary, not a drop-in script or redistributed game source.

1. Create a hierarchy through worldAPIs.editors:CreateUndoOperationsHierarchy().
2. CreateNewPart(resourceName, requestToken); await api.entity.HaveRequestsCompleted(requestToken).
3. StartEditingPart_Scenery(createObject) returns an edit/move object.
4. Read GetTransform/GetQuaternion/GetScale. Read vector coordinates with GetX/GetY/GetZ.
5. Construct the desired transform with TransformQ.FromOrPos(quaternion, Vector3:new(x,y,z)); SetTransform on the edit object.
6. Capture part IDs using worldAPIs.scenery:GetMoveObjectPartIDs(editObject).
7. PreviewChangeList(hierarchy:GetFullUndoChangeList()) returns operation-wait and preview tokens. Wait for IsOperationComplete(waitToken).
8. Wait for CanCommitPreview(previewToken) to resolve (nil was observed while pending); true permits the tested commit path.
9. CommitPreview(previewToken) exactly once, then await its operation completion and idle/no-preview state. Checkpoint({}) is an undo checkpoint only.
10. Check part existence, resource, primary/all entity mappings and world transform over separate component Advances.

Existing-part editing uses hierarchy:EditExistingPart(partID, nil, requestToken), waits completion and obtains StartEditingPart_Scenery's edit object. Deletion uses editObject:Destroy() followed by the same preview/commit machinery. A completed CancelPreview restores the tested world state; it need not restore the identical EntityID.

World checks use PlacementPartExists, GetPartResourceName, PlacementIDToEntityID and PlacementIDToAllEntityIDs on the tested placement API. World position is read via api.transform.CalculateWorldTransform(entityID):GetPos(). Mod-owned keys, part IDs and entity IDs are different domains.

## Persistence procedure

019/022/025 separated mutation/save and verification into different runs. Before manually loading the new save, rebuild Main.ovl with verify-only code: no creation, movement or deletion during load verification.

The inspected and tested save path obtains Interfaces.IParkLoadSaveManager, generates current metadata and calls SaveParkToSaveToken using the game owner plus a new local-save descriptor. Await manager completion and api.save.IsBusy() becoming false; validate the returned save token/name. Never retry an uncertain commit/save blindly.

Part 1182 is an example from this test save, not a globally stable ID. It identified AQ_Rock_04 at (10,5,10), then (15,5,10), and was absent after saved deletion.

## Command safety and comparison

028–030 use mod-defined object keys stone-A / stone-B and command IDs with ordered sequence numbers. Reserve a command before mutation, reject same-ID/different-content, acknowledge completed duplicates without replay, and verify world effects before marking completion. These tests provide session-local at-most-once commit invocation for the tested inputs, not distributed exactly-once processing.

030's twelve inputs include four valid operations, four order rejections and four ID conflicts. Each run ends with received=12, applied=4, rejected=8, orderErrors=4, idConflicts=4, commits=4, snapshots=12 and nextSequence=5. Canonical records compare logical keys, normalized resource, positions, mapping presence and counts; they do not require local entity IDs to match.

## Packaging controls

The component callback is Advance, not OnAdvance. Coroutines were resumed once per component Advance for cross-update checks. A world transition may recreate the component rather than preserve the same Lua object.

Several attempted runs used stale or missing Main.ovl. A loose Lua edit is not evidence of packed runtime code: regenerate the archive, replace the intended copied installation's archive, restart fully and verify the diagnostic/version marker before interpreting results. Mark those earlier attempts invalid, not failures of the intended diagnostic.
