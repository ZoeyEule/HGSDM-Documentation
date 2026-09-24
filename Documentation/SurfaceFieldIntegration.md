# Surface field integration — contract v2

Ordinary SSDM use requires only the existing WildMods SSDM component and a Profile.
No field actor, second component, interest, or CPU query is required to render relief.
Core's optional field foundation becomes active only for explicitly requested work.
Future module-specific area actors and gameplay behavior belong to their separately
installed, separately sold modules; they are not Core setup.

This reference targets the public headers in this source build, Unreal Engine 5.8,
`WildModsSSDM::SurfaceField::ContractVersion == 2`. Recompile consumers against the
matching Core headers and binaries when upgrading: this is a source integration
contract, not a promise that arbitrary C++ struct layouts remain binary compatible.
Stable enum/wire values are append-only. The tile archive version is separately 1.

## Obtain a subsystem and register a domain

All public subsystem/registry/owner calls below run on the game thread. Obtain
`World->GetSubsystem<UWildModsSSDMWorldSubsystem>()`; it can be null for unsupported
world types. Never retain a raw world/subsystem pointer through teardown.

The existing registered `UWildModsSSDMComponent` publishes its domain automatically.
Its serialized `SurfaceDomainId` is stable; ordinary duplication creates a new ID
and PIE duplication preserves the editor ID. Configure the component's
`SurfaceFieldDomainMode`: `WorldGround` or `TargetLocalPlane` (the latter requires
one explicit target primitive). Calling the normal `RegisterComponent()` after
assigning `Profile` and `TargetPrimitive` registers the source/domain. Use
`GatherFieldDomains(TArray<FWildModsSSDMSurfaceDomain>&) const` to obtain its frame,
bounds and source revision. There is no separate public RegisterDomain call.

A custom provider implements every method of
`IWildModsSSDMSurfaceFieldProvider`, returns its domains from
`GatherSurfaceFieldDomains(TArray<FWildModsSSDMSurfaceDomain>&) const`, and registers
with `FWildModsSSDMSurfaceFieldRegistry::Register(UWorld*, IWildModsSSDMSurfaceFieldProvider*)`.
Pair this with `Unregister(UWorld*, IWildModsSSDMSurfaceFieldProvider*)` before the
provider is destroyed. The registry does not own that pointer. Never reuse an old
capture across provider unregister/re-register, even if IDs and revisions match.
The v2 provider interface is declared completely in
[WildModsSSDMSurfaceField.h](../Source/WildMods_SSDM/Public/WildModsSSDMSurfaceField.h).
External metadata-only capture providers do not automatically gain GPU output
delivery; the complete GPU producer example below uses Core's own source capture.

Ground domains and planar vertical domains are supported. The independent raster
capture supports opaque/masked, non-Nanite local-vertex-factory static mesh sources.
Nanite, spline/other unsupported factories, dynamic-only geometry, translucent
materials, World Position Offset and Pixel Depth Offset fail with a reason.
Arbitrary curved charts, closed shells, depth peeling and multi-surface volumes
are not supported. These field restrictions do not redefine the ordinary screen
renderer's supported workflows.

## Interests, views and layers

Exact registration signatures:

```cpp
FWildModsSSDMFieldInterestHandle RegisterFieldInterest(
    UObject* Owner, const FWildModsSSDMFieldInterestDescriptor& Descriptor);
bool UpdateFieldInterest(FWildModsSSDMFieldInterestHandle Handle,
    const FWildModsSSDMFieldInterestDescriptor& Descriptor);
void UnregisterFieldInterest(FWildModsSSDMFieldInterestHandle Handle);
FWildModsSSDMFieldLayerHandle RegisterFieldLayer(UObject* Owner,
    const FWildModsSSDMFieldLayerDescriptor& Descriptor,
    IWildModsSSDMFieldLayerProducer* Producer);
void UnregisterFieldLayer(FWildModsSSDMFieldLayerHandle Handle);
```

Owners are weak; stop/unregister explicitly before destroying a producer.
Registration retains metadata and does not allocate a tile immediately. Scheduling
expands bounded domain ranges into shared sparse tiles. Bounds used for allocation
are half-open; tile coordinates use floor division, including negative coordinates.
Request a texel size at least the domain's persistent size for the Persistent tier;
finer requests select LocalDetail, subject to the configured resolution cap.
Persistence is an explicit policy, independent of the tier name.
`bRequiresCPUAccess` is metadata and never automatically issues readback.

| Purpose (`EViewPurpose`) | Result |
| --- | --- |
| `BaseSurface` | Captured material/Profile height; bypasses layer composition. |
| `VisibleSurface` | Layers with VisibleSurface capability. |
| `PhysicalSolid` | Layers with PhysicalSolid capability; no collision generation. |
| `BedObstacle` | Layers with BedObstacle capability; no water simulation. |
| `ExposureReceiver` | First eligible receiver along the oriented projection, including registered overlapping roof/floor domains. |
| `RawLayer` | One explicit `RawLayerId`, without composing its height against the base. |

Visible/physical/bed tiles use `BaseSurfaceLayerId` in their key. Raw tiles use
their module's stable layer ID and schema. Do not derive gameplay meaning from
the word "visible": field composition does not automatically draw module layers
through the existing screen-space renderer. Consumers implement that integration.

Declare Height **and Coverage** on layer outputs and request the channels actually
needed. Height is R32F in centimetres along the domain height axis; captured height
is absolute domain height. Coverage is R8_UINT, 0..255. Optional SurfaceId is
R32_UINT; Payload0/1 are RGBA16F. AddSigned/AddPositive take offsets/thickness;
MaskedMax/MaskedReplace take absolute height. Priority followed by stable GUID
orders contributors deterministically. Capability flags filter before producer
calls. Base coverage is preserved in composed views; Raw keeps raw coverage.
Core's four operations are generic, not snow/sand/water behavior.

A producer provides `uint64 GetFieldLayerRevision() const` and
`TSharedRef<IWildModsSSDMFieldLayerRenderProxy, ESPMode::ThreadSafe>
CreateFieldLayerRenderProxy() const` on the game thread. Only the immutable proxy
runs on the render thread. Its exact signature appears in the example.
Never capture a UObject, subsystem, producer pointer, or mutable owner state in it.
Shared base resources may be finer than the requested output extent. Sample with
normalized tile coordinates. Core reconciles layouts using cell-centre nearest
sampling for all channels together, not area averaging or conservative maxima.

## GPU consumption and direct capture

```cpp
bool GetFieldTileRenderResources(const FWildModsSSDMFieldTileKey& Key,
    FWildModsSSDMSurfaceCaptureRenderResources& OutResources) const;
bool GetFieldTileDiagnostic(const FWildModsSSDMFieldTileKey& Key,
    FWildModsSSDMFieldTileDiagnostic& OutDiagnostic) const;
FWildModsSSDMSurfaceCaptureHandle RequestSurfaceCapture(
    const FWildModsSSDMSurfaceCaptureRequest& Request);
bool GetSurfaceCapture(FWildModsSSDMSurfaceCaptureHandle Handle,
    FWildModsSSDMSurfaceCaptureDescriptor& OutDescriptor) const;
bool GetSurfaceCaptureRenderResources(FWildModsSSDMSurfaceCaptureHandle Handle,
    FWildModsSSDMSurfaceCaptureRenderResources& OutResources) const;
bool CancelSurfaceCapture(FWildModsSSDMSurfaceCaptureHandle Handle);
void ReleaseSurfaceCapture(FWildModsSSDMSurfaceCaptureHandle Handle);
```

Copy pooled refs on the game thread, then capture that owned value in the consumer's
render command. Register each texture with that graph's `RegisterExternalTexture`.
Never retain RDG refs across graphs. Copies may physically outlive invalidation;
check current revisions/status when acquiring new work and stop using obsolete
snapshots for authoritative decisions. Returning false never means "zero height".

A direct capture is also explicit work and can operate without a registered
interest. Supply valid world/domain bounds, extent, channels, projection and
provider/domain IDs. `FirstEligibleHit` selects the first surviving eligible
receiver; `AllEligible` is the selected single-domain capture, not depth peeling.
The descriptor identifies source and completed consumer revisions. Poll without
blocking; cancel Pending requests and always release retained handles.

## Explicit CPU access and persistence

```cpp
FWildModsSSDMFieldReadbackHandle RequestFieldTileCPUData(
    const FWildModsSSDMFieldTileKey& Key,
    WildModsSSDM::SurfaceField::EChannel Channels,
    WildModsSSDM::SurfaceField::EInterestPriority Priority);
FWildModsSSDMFieldReadbackHandle RequestPersistentFieldTileCPUData(
    const FWildModsSSDMFieldTileKey& Key,
    WildModsSSDM::SurfaceField::EChannel Channels,
    WildModsSSDM::SurfaceField::EInterestPriority Priority,
    uint64 ExpectedRevision = 0, FString* OutReason = nullptr);
bool PollFieldTileCPUData(FWildModsSSDMFieldReadbackHandle Handle,
    FWildModsSSDMFieldCPUData& OutData,
    WildModsSSDM::SurfaceField::ERequestStatus& OutStatus);
bool CancelFieldTileCPUData(FWildModsSSDMFieldReadbackHandle Handle);
void GatherDirtyPersistentTiles(TArray<FWildModsSSDMFieldTileKey>& OutKeys) const;
bool SerializeFieldTile(const FWildModsSSDMFieldCPUData& Data,
    TConstArrayView<uint8> OwnerMetadata, FWildModsSSDMSerializedFieldTile& Out,
    FString* OutReason = nullptr) const;
bool RestoreFieldTile(const FWildModsSSDMSerializedFieldTile& Tile,
    FString* OutReason = nullptr);
bool ClearPersistentTileDirtyFlag(const FWildModsSSDMFieldTileKey& Key,
    uint64 SavedRevision);
```

Readbacks are explicit asynchronous snapshots of already available channels.
They do not allocate tiles or enlarge channel sets. Pending polls retain the
handle; a terminal poll consumes it. Cancelled, Failed, Stale, Invalid and Ready
are distinct. The queue is bounded to 1024 retained handles and uses priority/FIFO
with a per-frame submission budget. Consume terminal results and retry deliberately.
A stale GPU completion cannot overwrite newer tile/source/layer generations.

Ordinary CPU lookup prefers current Ready data, with the existing retired
persistent fallback. Always use the persistent selector when saving: a newer
transient view can coexist with an older dirty persistent snapshot.
ExpectedRevision 0 selects the retained snapshot at admission; a nonzero value
requires an exact match. No fallback to transient data occurs. Clearing or
replacing that snapshot before delivery reports Stale.
`bRetainedPersistentData` is save-only runtime metadata, not a wire flag.

Serialize only owned dirty state after a Ready CPU result. Serialize does not
clear dirtiness. Persist the archive successfully, then clear using the exact
CPU revision. If a newer edit wins, clear returns false and dirty state remains.
Archives provide `bool Serialize(FArchive&, uint32 MetadataLimit =
DefaultOwnerMetadataBytes)`; loading validates bounded lengths, versions, stable
channel formats, finite data and CRC before mutation. Owner metadata defaults to
64 KiB (hard ceiling 1 MiB); data is capped at 128 MiB, extents at 2048 each.
CRC detects accidental corruption, not hostile tampering or authentication.
The owner chooses disk location, async I/O, durability and metadata semantics.

Restore needs the current domain/provider/layer/schema registrations and normal
RHI. Admission validates layout/channels/revision and count/byte budgets. A true
return means accepted, not Ready. Observe the tile diagnostic while the upload
completes. Current-session source/registration identities are bound at admission;
retired or replaced providers/layers cannot publish old uploads. Failed upload
can retry the same archive, but a newer successful persistent commit rejects old
archives. Defaults are 8 outstanding uploads and 64 MiB of encoded/archive-owner
bytes, including superseded submissions until completion/cancellation retirement.

## Dirty regions, stopping and diagnostics

```cpp
void MarkSurfaceFieldDirty();
void MarkSurfaceFieldDirty(const FBox3d& WorldRegion);
void MarkFieldLayerDirty(FWildModsSSDMFieldLayerHandle Handle,
    const FBox2d& DomainRegion);
FWildModsSSDMFieldStats GetFieldStats() const;
void DumpFieldStats(FOutputDevice& Output) const;
```

Base dirty calls affect Core-owned sources. External providers advance their own
domain/source revisions. World dirty bounds are closed: an exact shared tile
edge invalidates both neighbours. Layer dirty bounds are domain coordinates;
only contributing views are invalidated. Advance a producer's revision whenever
its immutable content changes. Explicit notifications are needed for in-place
data edits that preserve object identities.

Cancel CPU/direct requests, unregister interests and layers, and unregister
custom providers before owner/module shutdown. Save-required state must be
durably saved before world destruction. Owner retirement and idle release retain
dirty snapshots as save-only data; world teardown releases them. Mark a layer
dirty before unregistering it to revoke retained composition work immediately.
A render proxy can outlive its game-thread producer; a module containing its
virtual methods must remain loaded until queued render work and owned proxies
retire. Runtime Stop need not stall. At an actual module unload, quiesce callers,
stop sessions, and drain render commands before unloading code; disable dynamic
reload for a consumer module that cannot guarantee this lifetime.

`r.WildMods.SSDM.Field.Enabled 0` immediately cancels field capture, composition,
readback and restore work in all live worlds. Ordinary field GPU access is revoked;
transient ownership retires asynchronously. Dispatched Core probe components are
unregistered immediately and held alive until their render-retirement fences
poll complete; the disabled subsystem may tick only to finish that retirement.
Runtime bypass and idle release do not wait on those fences. Final world teardown
still drains outstanding probe retirement before releasing world-owned objects.
Unsaved persistent snapshots remain
charged and save-only. While disabled, new captures/readbacks/restores are
rejected; registration metadata remains. Re-enable to explicitly save retained
snapshots. Re-enabling without interest or another explicit request creates no
new work. This control never changes `r.WildMods.SSDM.Enabled` or the renderer
setting. Engine/consumer-owned refs may physically retire later.

Run `WildMods.SSDM.Field.Dump` for the invoking world, or append `all` for every
live editor/PIE/game world. No automatic per-tick log is added. The copied
game-thread stats never wait for GPU completion. Provider/domain counts include
registered sources even when inactive; Core alone contributes one provider.
Tile counts, captures, compositions, readbacks, restores and bytes remain zero
in a fresh renderer-only world. Tile state/tier rows include stale records;
Stale is separated from actionable failures. RecentFailure lists up to eight
retained actionable failures, globally newest first across tile and direct-capture
failures using one monotonic game-thread observation sequence. It does not compare
unrelated source/consumer revisions or refresh recency when dumped. It is not a
permanent history after records are released.

ResourceBytes is the tile store's accounted pooled output ownership, including
dirty retained snapshots. Shared current/persistent ownership of one tile is
charged once; an independently replaced output is charged separately. This is
the existing tile budget metric, not total GPU VRAM: transient RDG targets,
source banks, readback staging, driver pools and refs held by consumers are
excluded. RestoreUploadBytes is separately the admitted archive byte charge.
Do not infer total memory/performance from the tile budget or claim a GPU fence
has completed from these counts. Unsupported source/layer/capture diagnostics
fail closed, with key, cause, generation and corrective reason.

Core supplies no built-in multiplayer transport, replication, authority model,
bandwidth policy or update frequency. A game/module owns those decisions and
must validate remote data under its own trust and ownership rules.

## Complete separate-module example

The following example uses public Core headers without Renderer dependencies or private Core include paths.

In your own module, add `Core`, `CoreUObject`, `Engine`, `RHI`, `RenderCore`,
and `WildMods_SSDM` to the appropriate Build.cs dependency list; declare the
Core plugin dependency in your .uplugin. Own one session per chosen surface/world.
Call Start with a registered component, GetGPU from your game-thread update,
RequestSave/PollSave when persistence is desired, and Stop before its owner ends.
The example uses no implementation-private helpers.

The example below is included directly here for separate-module integration.
It requests one bounded persistent VisibleSurface tile and adds a constant 5 cm
typed layer. The example does not change screen rendering, simulate gameplay,
choose a file path or create network transport.

```cpp
#pragma once
#include "WildModsSSDMComponent.h"
#include "WildModsSSDMWorldSubsystem.h"
#include "WildModsSSDMFieldRender.h"
#include "Engine/World.h"
#include "RenderGraphBuilder.h"
#include "RenderGraphUtils.h"

// A module-owned session. Construct/use/destroy on the game thread.
// The module must outlive all render proxies it creates.
class FSurfaceFieldExample final : public IWildModsSSDMFieldLayerProducer
{
 using EChannel=WildModsSSDM::SurfaceField::EChannel;
 using EStatus=WildModsSSDM::SurfaceField::ERequestStatus;
 class FProxy final : public IWildModsSSDMFieldLayerRenderProxy
 {
  const float HeightCm;
 public:
  explicit FProxy(float H):HeightCm(H){}
  void AddFieldLayerPasses_RenderThread(FRDGBuilder& Graph,
   const FWildModsSSDMFieldLayerRenderContext& Context,
   FWildModsSSDMFieldRenderTargets& Out) override
  {
   check(IsInRenderingThread());
   // AddSigned is an offset in centimetres. Coverage is unsigned 0..255.
   AddClearUAVPass(Graph,Out.Height,HeightCm);
   AddClearUAVPass(Graph,Out.Coverage,255u);
  }
 };
 TWeakObjectPtr<UWildModsSSDMWorldSubsystem> Subsystem;
 FWildModsSSDMFieldInterestHandle Interest;
 FWildModsSSDMFieldLayerHandle Layer;
 FWildModsSSDMSurfaceDomain Domain;
 FWildModsSSDMFieldTileKey Key;
 TArray<FWildModsSSDMFieldReadbackHandle> PendingSaves;
 uint64 ProducerRevision=1;
 float OffsetCm=5.f;
public:
 ~FSurfaceFieldExample(){Stop();}
 uint64 GetFieldLayerRevision() const override
 {check(IsInGameThread());return ProducerRevision;}
 TSharedRef<IWildModsSSDMFieldLayerRenderProxy,ESPMode::ThreadSafe>
 CreateFieldLayerRenderProxy() const override
 {check(IsInGameThread());return MakeShared<FProxy,ESPMode::ThreadSafe>(OffsetCm);}

 bool Start(UWildModsSSDMComponent& Surface)
 {
  check(IsInGameThread());Stop();
  UWorld* World=Surface.GetWorld();
  auto* S=World?World->GetSubsystem<UWildModsSSDMWorldSubsystem>():nullptr;
  if(!S||!Surface.IsRegistered()||S->GetSurfaceFieldContractVersion()!=2)return false;
  TArray<FWildModsSSDMSurfaceDomain> Domains;S->GatherFieldDomains(Domains);
  const auto* D=Domains.FindByPredicate([&](const auto& Item)
   {return Item.DomainId==Surface.SurfaceDomainId;});
  if(!D||!D->IsValid())return false;
  Domain=*D;Subsystem=S;
  Key={};Key.DomainId=Domain.DomainId;
  Key.LayerId=WildModsSSDM::SurfaceField::BaseSurfaceLayerId;
  Key.ViewPurpose=WildModsSSDM::SurfaceField::EViewPurpose::VisibleSurface;
  Key.ResolutionTier=WildModsSSDM::SurfaceField::EResolutionTier::Persistent;
  Key.Coordinate=FIntPoint(FMath::FloorToInt(Domain.LocalBounds.Min.X/Domain.TileWorldSizeCm),
   FMath::FloorToInt(Domain.LocalBounds.Min.Y/Domain.TileWorldSizeCm));
  const FVector2d TileMax=FVector2d(Key.Coordinate+FIntPoint(1))*Domain.TileWorldSizeCm;
  FWildModsSSDMFieldLayerDescriptor L;
  L.DomainId=Domain.DomainId;L.LayerId=FGuid(0x73A61A11,0xDB774A22,0x8C715B33,0xBE537C44);
  L.DebugName=TEXT("IntegrationExampleOffset");L.Channels=WildModsSSDM::SurfaceField::CoreRenderChannels;
  L.RequiredBaseChannels=L.Channels;
  L.Capabilities=EWildModsSSDMFieldLayerCapability::VisibleSurface|EWildModsSSDMFieldLayerCapability::Persistent;
  L.HeightOperation=EWildModsSSDMFieldCompositionOp::AddSigned;
  L.PersistencePolicy=EWildModsSSDMFieldPersistencePolicy::SaveRequired;
  Layer=S->RegisterFieldLayer(&Surface,L,this);
  if(!Layer.IsValid()){Stop();return false;}
  FWildModsSSDMFieldInterestDescriptor I;I.DomainId=Domain.DomainId;
  I.DomainBounds=FBox2d(Domain.LocalBounds.Min,
   FVector2d(FMath::Min(TileMax.X,Domain.LocalBounds.Max.X),FMath::Min(TileMax.Y,Domain.LocalBounds.Max.Y)));
  I.RequestedTexelSizeCm=Domain.PersistentTexelSizeCm;
  I.RequestedChannels=L.Channels;I.ViewPurpose=Key.ViewPurpose;I.bRequiresPersistence=true;
  Interest=S->RegisterFieldInterest(&Surface,I);
  if(!Interest.IsValid()){Stop();return false;}return true;
 }
 // Copy refs on the game thread; register them as external textures in the
 // consumer's own render graph. Never store an FRDGTextureRef between graphs.
 bool GetGPU(FWildModsSSDMSurfaceCaptureRenderResources& Out) const
 {check(IsInGameThread());auto* S=Subsystem.Get();return S&&S->GetFieldTileRenderResources(Key,Out);}
 static FRDGTextureRef RegisterHeight_RenderThread(FRDGBuilder& Graph,
  const FWildModsSSDMSurfaceCaptureRenderResources& Owned)
 {check(IsInRenderingThread());return Owned.Height?Graph.RegisterExternalTexture(Owned.Height):nullptr;}

 void SetOffset(float Value)
 {
  check(IsInGameThread());if(!FMath::IsFinite(Value)||Value==OffsetCm)return;
  OffsetCm=Value;++ProducerRevision;
  if(auto* S=Subsystem.Get();S&&Layer.IsValid())S->MarkFieldLayerDirty(Layer,Domain.LocalBounds);
 }
 void DirtyBase(const FBox3d& WorldRegion)
 {check(IsInGameThread());if(auto* S=Subsystem.Get())S->MarkSurfaceFieldDirty(WorldRegion);}
 void RequestSave()
 {
  check(IsInGameThread());auto* S=Subsystem.Get();if(!S||!PendingSaves.IsEmpty())return;
  TArray<FWildModsSSDMFieldTileKey> Dirty;S->GatherDirtyPersistentTiles(Dirty);
  // This example owns only this composed tile; never save another module's data.
  if(Dirty.Contains(Key))
  {
   auto H=S->RequestPersistentFieldTileCPUData(Key,WildModsSSDM::SurfaceField::CoreRenderChannels,
    WildModsSSDM::SurfaceField::EInterestPriority::Background,0);
   if(H.IsValid())PendingSaves.Add(H);
  }
 }
 // Call from the owner's game-thread tick. The callback must return true only
 // after durable storage succeeds. Returning false leaves Core's dirty flag set.
 void PollSave(TFunctionRef<bool(const FWildModsSSDMSerializedFieldTile&)> CommitArchive)
 {
  check(IsInGameThread());auto* S=Subsystem.Get();if(!S)return;
  for(int32 N=PendingSaves.Num()-1;N>=0;--N)
  {
   FWildModsSSDMFieldCPUData CPU;EStatus Status=EStatus::Invalid;
   const bool Known=S->PollFieldTileCPUData(PendingSaves[N],CPU,Status);
   if(Known&&Status==EStatus::Pending)continue;
   PendingSaves.RemoveAt(N);
   if(!Known||Status!=EStatus::Ready)continue; // Owner may request a fresh save later.
   FWildModsSSDMSerializedFieldTile Archive;FString Reason;
   if(S->SerializeFieldTile(CPU,TConstArrayView<uint8>(),Archive,&Reason)&&CommitArchive(Archive))
    S->ClearPersistentTileDirtyFlag(CPU.Key,CPU.Revision); // False means a newer edit won.
  }
 }
 bool Restore(const FWildModsSSDMSerializedFieldTile& Archive,FString& Reason)
 {check(IsInGameThread());auto* S=Subsystem.Get();return S&&Archive.Key==Key&&S->RestoreFieldTile(Archive,&Reason);}
 void Stop()
 {
  check(IsInGameThread());
  if(auto* S=Subsystem.Get();S&&S->GetFieldStats().ProviderCount>0)
  {
   for(auto H:PendingSaves)S->CancelFieldTileCPUData(H);
   // Drop cancelled terminal handles as well.
   for(auto H:PendingSaves){FWildModsSSDMFieldCPUData D;EStatus Status;S->PollFieldTileCPUData(H,D,Status);}
   if(Layer.IsValid())
   {
    // Revoke retained view/proxy work before destroying this producer.
    S->MarkFieldLayerDirty(Layer,Domain.LocalBounds);
    S->UnregisterFieldLayer(Layer);
   }
   if(Interest.IsValid())S->UnregisterFieldInterest(Interest);
  }
  PendingSaves.Reset();Layer.Reset();Interest.Reset();Subsystem.Reset();
 }
};
```
