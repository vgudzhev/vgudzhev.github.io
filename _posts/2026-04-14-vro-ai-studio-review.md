---
title: "vRO AI Studio: Code Review That Knows Your Rules"
date: 2026-04-14 12:00:00 +0200
categories: [VMware, AI]
tags: [vro, aria-orchestrator, vscode, claude, typescript, automation, code-review]
---

Not all code starts with standards. Some was written before you had them. Some was written by someone who didn't know them. Some was written at 11pm before a deployment window.

Review handles all three cases.

## The Problem with Code Review in vRO

Human reviewers have to hold a lot of context. Does `i` mean a VM index or a retry counter? Is the error being swallowed or logged? Is this SDK call going to pull 10,000 VMs into memory?

The reviewer has to know the runtime, the SDK behavior, and your team's conventions — all at once.

Things get missed. Always. The question is which things, and how expensive are they to find later.

## What Happens When You Feed Your Standards to the Review Tab

When you load your `vro-standards.md` into vRO AI Studio alongside your action, the Review isn't just checking for generic issues. It knows your rules. It flags violations by name.

Here's a real example. Action written without context:

```typescript
export function getVms(tag: string): Properties {
  var result = new Properties();
  var i = 0;
  var vms = VcPlugin.getAllVirtualMachines();
  for (i = 0; i < vms.length; i++) {
    try {
      if (vms[i].runtime.powerState === VcVirtualMachinePowerState.poweredOff) {
        result.put(vms[i].name, vms[i].config.hardware.memoryMB);
      }
    } catch(e) {}
  }
  return result;
}
```

Run Review with your standards loaded. Findings:

- **Error** — `VcPlugin.getAllVirtualMachines()` is a blocking synchronous call. Retrieves full inventory before filtering. Use vCenter search APIs with filters instead.
- **Warning** — Loop index named `i`. Your standards require meaningful names — `vmIndex` or `vmItem`.
- **Warning** — Empty catch block. Errors are silently swallowed. Standards require `System.error()` with input values before rethrowing.
- **Warning** — No `System.log()` at function entry. Standards require logging the start of every action with its inputs.
- **Info** — Boolean condition on power state uses raw SDK enum. Consider extracting to a named helper `isPoweredOff(vm)` for readability.

The same action, written with the `vro-standards.md` from the start:

```typescript
export function getVmsByTag(tag: string): Properties {
  System.log("getVmsByTag started | tag: " + tag);

  if (!tag || tag.trim() === "") {
    throw new Error("tag parameter is required and cannot be empty");
  }

  var result = new Properties();

  try {
    var vms = VcPlugin.searchManagedEntities("VirtualMachine");

    for (var vmIndex = 0; vmIndex < vms.length; vmIndex++) {
      var vm = vms[vmIndex];
      try {
        if (isPoweredOff(vm) && hasTag(vm, tag)) {
          var vmName = vm.name || "Unknown";
          result.put(vmName, vm.config.hardware.memoryMB.toString());
          System.log("Matched VM: " + vmName);
        }
      } catch (vmError) {
        System.error("Error processing VM at index " + vmIndex + ": " + vmError);
      }
    }
  } catch (error) {
    System.error("getVmsByTag failed | tag: " + tag + " | " + error);
    throw error;
  }

  System.log("getVmsByTag complete | matched: " + result.keys.length);
  return result;
}
```

Same goal. Different quality. The Review tab tells you exactly what changed and why — citing your own standards back at you.

## What This Looks Like in Practice

The new engineer doesn't know the rules yet. Review does. They get the feedback before the PR — not during it. That's the difference.
