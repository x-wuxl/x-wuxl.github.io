---
layout: post
title: "解决 Claude Fable 5 因 Safety 机制自动回退至 Opus 4.8 的方法"
date: 2026-06-10
last_modified_at: 2026-06-10
categories: [AI, Tools]
tags: [Claude, Fable 5, Claude Code, Troubleshooting, PowerShell, Web Development]
keywords: [
  CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK,
  Claude Fable 5,
  Opus 4.8 fallback,
  Switched to Opus 4.8,
  Fable 5's safety measures flagged this message,
  Claude Fable 5 自动回退,
  Claude Code safety fallback,
  Claude Code settings.json env,
  Claude 拒绝回答自动降级,
  如何关闭 Claude Fable 安全回退
]
description: "本文介绍如何通过配置环境变量，解决 Claude Fable 5（Claude Code 高阶模型）因安全机制误判（如网络安全或生物学话题）而自动回退到较旧 Opus 4.8 模型的问题。"
author: "wuxl"
excerpt: "Claude Fable 5 在处理涉密或敏感话题时经常误判并自动回退至较旧的 Opus 4.8。本文分享通过配置 settings.json 的 env 选项禁用此回退机制，确保正常使用高阶模型。"
toc: true
toc_sticky: true
---

> Fable 5's safety measures flagged this message for cybersecurity or biology topics. They may flag safe, normal content as well. These measures let us bring you Mythos-level capability in other areas sooner, and we're working to refine them. Switched to Opus 4.8. Send feedback with /feedback or learn more.

**Fable 5** 是 Anthropic 推出的 Mythos-class 模型（Claude 的高阶版本），面向公众可用，但内置了强安全分类器（classifiers）。

当查询被判定涉及**高风险领域**（cybersecurity、网络安全、biology/chemistry 生物化学、model distillation 等）时，它不会直接拒绝，而是**自动回退（fallback）到较老/较弱的 Opus 4.8** 来生成回复，同时显示以上提示。

分类器目前设置得比较宽松/保守，经常误判正常内容（如代码项目、研究、甚至无关话题）

### 解决方案

以下代码用来解决 Claude Fable 5 因 safety 原因自动回退到 Opus 4.8 的问题

#### 第一步：修改配置文件

先在 `~/.claude/settings.json`  配置文件 `env` 配置项中添加

```
"CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK":"1"
```

#### 第二步：根据系统运行配置脚本

***windows*** 系统，新建一个 **`.ps1`** 格式的文件（例如：**fix_claude_fallback.ps1**）复制以下代码进行执行

```
<#
.SYNOPSIS
    Claude Code Refusal Fallback No-Switch Fix Script (Windows Version)

.DESCRIPTION
    THE BUG:
    CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK env var kills the entire refusal
    fallback mechanism - the turn stops dead on safety refusal, with no retry
    and no continuation. Users who don't want model switching have no way to
    keep the turn alive.

    ROOT CAUSE:
    ed() returns false when the env var is set, which short-circuits ALL
    fallback logic: no server-side fallback signalling, no client-side retry,
    no dialog prompt. The turn just dies on stop_reason "refusal".

    FIX POINTS:
    1) ed() -> always return true (fallback machinery stays active)
    2) swapSession block -> add env var guard so model is NOT persisted when set
       The server's fallback response is still accepted (turn completes), but
       mainLoopModel / mainLoopModelForSession / mainLoopModelOverride stay
       unchanged - next turn uses the user's original model.

    Applicable: v2.1.170+ (requires swapSession/latch architecture)

.PARAMETER Check
    Check if fix is needed without making changes

.PARAMETER Restore
    Restore original file from backup

.PARAMETER Help
    Show help information

.PARAMETER CliPath
    Path to cli.js file (optional, auto-detect if not provided)

.EXAMPLE
    .\apply-claude-code-refusal-no-switch-fix.ps1
    Apply the fix (auto-detect cli.js location)

.EXAMPLE
    .\apply-claude-code-refusal-no-switch-fix.ps1 -CliPath "C:\path\to\cli.js"
    Apply fix to specific file

.EXAMPLE
    .\apply-claude-code-refusal-no-switch-fix.ps1 -Check
    Check if fix is needed

.EXAMPLE
    .\apply-claude-code-refusal-no-switch-fix.ps1 -Restore
    Restore from backup

.NOTES
    Requirements:
    - Node.js (already installed if you have Claude Code)
    - Internet connection (downloads acorn parser on first run)

    Note: This patch will be overwritten when Claude Code updates.
    Re-run this script after updates if the issue reappears.
#>

param(
    [switch]$Check,
    [switch]$Restore,
    [switch]$Help,
    [string]$CliPath
)

# ============================================================
# Configuration
# ============================================================
$BACKUP_SUFFIX = "backup-refusal-no-switch"
$FIX_DESCRIPTION = "Change CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK to allow turn completion without model switch"

# ============================================================
# Color output functions
# ============================================================
function Write-Success { param($Message) Write-Host "[OK] " -ForegroundColor Green -NoNewline; Write-Host $Message }
function Write-Warning { param($Message) Write-Host "[!] " -ForegroundColor Yellow -NoNewline; Write-Host $Message }
function Write-FixError { param($Message) Write-Host "[X] " -ForegroundColor Red -NoNewline; Write-Host $Message }
function Write-Info { param($Message) Write-Host "[>] " -ForegroundColor Blue -NoNewline; Write-Host $Message }

# ============================================================
# Main function - Wrapped to avoid exit closing terminal via iex
# ============================================================
function Invoke-ClaudeCodeFix {
    param(
        [switch]$Check,
        [switch]$Restore,
        [switch]$Help,
        [string]$CliPath
    )

    # Show help
    if ($Help) {
        Write-Host @"
Claude Code $FIX_DESCRIPTION

Usage:
    .\$($MyInvocation.MyCommand.Name) [options]

Options:
    -Check      Check if fix is needed without making changes
    -Restore    Restore original file from backup
    -CliPath    Path to cli.js file (optional, auto-detect if not provided)
    -Help       Show this help message

Examples:
    .\$($MyInvocation.MyCommand.Name)
    .\$($MyInvocation.MyCommand.Name) -Check
    .\$($MyInvocation.MyCommand.Name) -CliPath "C:\path\to\cli.js"
"@
        return 0
    }

    # --------------------------------------------------------
    # Find Claude Code cli.js path
    # --------------------------------------------------------
    function Find-CliPath {
        $locations = @(
            (Join-Path $env:USERPROFILE ".claude\local\node_modules\@anthropic-ai\claude-code\cli.js"),
            (Join-Path $env:APPDATA "npm\node_modules\@anthropic-ai\claude-code\cli.js"),
            (Join-Path $env:ProgramFiles "nodejs\node_modules\@anthropic-ai\claude-code\cli.js"),
            (Join-Path $env:USERPROFILE ".claude\local\node_modules\@cometix\claude-code\cli.js"),
            (Join-Path $env:APPDATA "npm\node_modules\@cometix\claude-code\cli.js"),
            (Join-Path $env:ProgramFiles "nodejs\node_modules\@cometix\claude-code\cli.js")
        )

        # Try to get global path from npm
        try {
            $npmRoot = & npm root -g 2>$null
            if ($npmRoot) {
                $locations += Join-Path $npmRoot "@anthropic-ai\claude-code\cli.js"
                $locations += Join-Path $npmRoot "@cometix\claude-code\cli.js"
            }
        } catch {}

        foreach ($path in $locations) {
            if (Test-Path $path) {
                return $path
            }
        }
        return $null
    }

    # --------------------------------------------------------
    # Determine cliPath: use provided path or auto-detect
    # --------------------------------------------------------
    if ($CliPath) {
        if (Test-Path $CliPath) {
            $cliPathResolved = $CliPath
            Write-Info "Using specified cli.js: $cliPathResolved"
        } else {
            Write-FixError "Specified file not found: $CliPath"
            return 1
        }
    } else {
        $cliPathResolved = Find-CliPath
        if (-not $cliPathResolved) {
            Write-FixError "Claude Code cli.js not found"
            Write-Host ""
            Write-Host "Searched locations:"
            Write-Host "  ~\.claude\local\node_modules\@anthropic-ai\claude-code\cli.js"
            Write-Host "  ~\.claude\local\node_modules\@cometix\claude-code\cli.js"
            Write-Host "  %APPDATA%\npm\node_modules\@anthropic-ai\claude-code\cli.js"
            Write-Host "  %APPDATA%\npm\node_modules\@cometix\claude-code\cli.js"
            Write-Host "  `$(npm root -g)\@anthropic-ai\claude-code\cli.js"
            Write-Host "  `$(npm root -g)\@cometix\claude-code\cli.js"
            Write-Host ""
            Write-Host "Tip: You can specify the path directly:"
            Write-Host "  .\$($MyInvocation.MyCommand.Name) -CliPath 'C:\path\to\cli.js'"
            return 1
        }
        Write-Info "Found Claude Code: $cliPathResolved"
    }

    $cliPath = $cliPathResolved

    # --------------------------------------------------------
    # Restore backup
    # --------------------------------------------------------
    if ($Restore) {
        $backups = Get-ChildItem -Path (Split-Path $cliPath) -Filter "cli.js.$BACKUP_SUFFIX-*" -ErrorAction SilentlyContinue |
                   Sort-Object LastWriteTime -Descending

        if ($backups.Count -gt 0) {
            $latestBackup = $backups[0].FullName
            Copy-Item $latestBackup $cliPath -Force
            Write-Success "Restored from backup: $latestBackup"
            return 0
        } else {
            Write-FixError "No backup file found (cli.js.$BACKUP_SUFFIX-*)"
            return 1
        }
    }

    Write-Host ""

    # --------------------------------------------------------
    # Download acorn parser if needed
    # --------------------------------------------------------
    $acornPath = Join-Path $env:TEMP "acorn-claude-fix.js"
    if (-not (Test-Path $acornPath)) {
        Write-Info "Downloading acorn parser..."
        try {
            Invoke-WebRequest -Uri "https://unpkg.com/acorn@8.16.0/dist/acorn.js" -OutFile $acornPath -UseBasicParsing
        } catch {
            Write-FixError "Failed to download acorn parser"
            return 1
        }
    }

    # --------------------------------------------------------
    # Node.js patch script
    # --------------------------------------------------------
    $patchScript = @'
const fs = require('fs');
const acornPath = process.argv[2];
const acorn = require(acornPath);

const cliPath = process.argv[3];
const checkOnly = process.argv[4] === '--check';

let code = fs.readFileSync(cliPath, 'utf-8');

// Preserve shebang
let shebang = '';
if (code.startsWith('#!')) {
    const idx = code.indexOf('\n');
    shebang = code.slice(0, idx + 1);
    code = code.slice(idx + 1);
}

// ============================================================
// Constants
// ============================================================
const ENV_VAR = 'CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK';
const PATCH_MARKER = 'CC_REFUSAL_NO_SWITCH_V1';

// Track fix status
let fixes = {
    edFunc: { found: false, patched: false, node: null },
    swapSession: { found: false, patched: false, node: null },
};

// Parse AST
let ast;
try {
    ast = acorn.parse(code, { ecmaVersion: "latest", sourceType: 'script' });
} catch (e) {
    try {
        ast = acorn.parse(code, { ecmaVersion: "latest", sourceType: 'module' });
    } catch (e2) {
        console.error('PARSE_ERROR:' + e2.message);
        process.exit(1);
    }
}

// AST helper: find nodes matching predicate
function findNodes(node, predicate, results = []) {
    if (!node || typeof node !== 'object') return results;
    if (predicate(node)) results.push(node);
    for (const key in node) {
        if (node[key] && typeof node[key] === 'object') {
            if (Array.isArray(node[key])) {
                node[key].forEach(child => findNodes(child, predicate, results));
            } else {
                findNodes(node[key], predicate, results);
            }
        }
    }
    return results;
}

// Get source code snippet from AST node
const src = (node) => code.slice(node.start, node.end);

// ============================================================
// Fix 1: Find ed() - the function that checks DISABLE_REFUSAL_FALLBACK
//
// Pattern A (v2.1.170):
//   function XX() { return !STORE.CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK }
//
// Pattern B (v2.1.162):
//   function XX() {
//     if (STORE.CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK) return !1;
//     return featureFlag(...)
//   }
// ============================================================

let edFunc = null;
let edStoreName = null;

const allFuncs = findNodes(ast, n =>
    (n.type === 'FunctionDeclaration' || n.type === 'FunctionExpression') &&
    n.params?.length === 0 &&
    n.body?.type === 'BlockStatement'
);

for (const fn of allFuncs) {
    const body = fn.body.body;

    // Pattern A: return !STORE.ENV_VAR
    if (body.length === 1 && body[0].type === 'ReturnStatement') {
        const arg = body[0].argument;
        if (arg?.type === 'UnaryExpression' && arg.operator === '!' &&
            arg.argument?.type === 'MemberExpression' &&
            arg.argument.property?.type === 'Identifier' &&
            arg.argument.property.name === ENV_VAR) {
            edFunc = fn;
            edStoreName = src(arg.argument.object);
            break;
        }
    }

    // Pattern B: if (STORE.ENV_VAR) return !1; return ...
    if (body.length === 2 && body[0].type === 'IfStatement' && body[1].type === 'ReturnStatement') {
        const test = body[0].test;
        if (test?.type === 'MemberExpression' &&
            test.property?.type === 'Identifier' &&
            test.property.name === ENV_VAR) {
            edFunc = fn;
            edStoreName = src(test.object);
            break;
        }
    }
}

if (edFunc) {
    const name = edFunc.id?.name || '<anon>';
    fixes.edFunc.found = true;
    fixes.edFunc.node = edFunc;
    console.log(`FOUND:edFunc "${name}" at byte ${edFunc.start} - checks ${ENV_VAR} on ${edStoreName}`);
}

// ============================================================
// Fix 2: Find swapSession block that persists model
//
// The if-test may be a SequenceExpression (comma operator):
//   if (W=!0, S=o_.toModel, wLH(), BH=o_.toModel, i_.swapSession) { ... }
//
// The last expression in the sequence is the actual boolean test:
//   MemberExpression: *.swapSession
//
// We wrap it: *.swapSession && !STORE.ENV_VAR
// ============================================================

const ifStmts = findNodes(ast, n => n.type === 'IfStatement');

for (const node of ifStmts) {
    let test = node.test;

    // Unwrap SequenceExpression - the real test is the last expression
    let swapExpr = null;
    if (test?.type === 'SequenceExpression' && test.expressions?.length > 0) {
        const last = test.expressions[test.expressions.length - 1];
        if (last?.type === 'MemberExpression' &&
            last.property?.type === 'Identifier' &&
            last.property.name === 'swapSession') {
            swapExpr = last;
        }
    }
    // Also handle simple case: if (*.swapSession) { ... }
    if (!swapExpr && test?.type === 'MemberExpression' &&
        test.property?.type === 'Identifier' &&
        test.property.name === 'swapSession') {
        swapExpr = test;
    }

    if (!swapExpr) continue;

    const block = node.consequent;
    if (!block) continue;
    const blockSrc = src(block);

    // Verify this is the model persistence block
    if (blockSrc.includes('mainLoopModel') &&
        blockSrc.includes('.toModel') &&
        blockSrc.includes('mainLoopModelForSession')) {
        fixes.swapSession.found = true;
        fixes.swapSession.node = { ifNode: node, swapExpr: swapExpr };
        console.log(`FOUND:swapSession block at byte ${node.start} (swap test at ${swapExpr.start})`);
        break;
    }
}

// ============================================================
// Check if already patched or patterns not found
// ============================================================

const allFound = Object.values(fixes).some(f => f.found);
if (!allFound) {
    const alreadyPatched = code.includes(PATCH_MARKER);
    if (alreadyPatched) {
        console.log('ALREADY_PATCHED');
        process.exit(2);
    }

    // Provide specific diagnostics
    if (!code.includes(ENV_VAR)) {
        console.error('NOT_FOUND:CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK not found in code (pre-v2.1.162?)');
    } else if (!code.includes('swapSession')) {
        console.error('NOT_FOUND:swapSession architecture not found (pre-v2.1.170?)');
    } else {
        console.error('NOT_FOUND:Unable to locate ed() function or swapSession block');
    }
    process.exit(1);
}

if (checkOnly) {
    console.log('NEEDS_PATCH');
    const count = Object.values(fixes).filter(f => f.found).length;
    console.log('PATCH_COUNT:' + count);
    process.exit(1);
}

// ============================================================
// Apply fixes using AST node positions
// ============================================================

let newCode = code;

function replaceAt(str, start, end, replacement) {
    return str.slice(0, start) + replacement + str.slice(end);
}

let replacements = [];

// Fix 1: ed() always returns true
if (fixes.edFunc.found && fixes.edFunc.node) {
    const node = fixes.edFunc.node;
    replacements.push({
        start: node.body.start,
        end: node.body.end,
        replacement: `{return!0/*${PATCH_MARKER}*/}`,
        name: 'edFunc'
    });
    fixes.edFunc.patched = true;
    console.log('PATCH:edFunc - ed() now always returns true (fallback machinery stays active)');
}

// Fix 2: swapSession guard - wrap the swapSession MemberExpression
if (fixes.swapSession.found && fixes.swapSession.node) {
    const { swapExpr } = fixes.swapSession.node;
    const swapSrc = src(swapExpr);
    const store = edStoreName || '$_';
    replacements.push({
        start: swapExpr.start,
        end: swapExpr.end,
        replacement: `${swapSrc}&&!${store}.${ENV_VAR}`,
        name: 'swapSession'
    });
    fixes.swapSession.patched = true;
    console.log(`PATCH:swapSession - model persistence guarded by !${store}.${ENV_VAR}`);
}

// Apply AST-based replacements from end to start to preserve positions
replacements.sort((a, b) => b.start - a.start);
for (const r of replacements) {
    newCode = replaceAt(newCode, r.start, r.end, r.replacement);
}

// ============================================================
// Verify and save
// ============================================================

const patchedCount = Object.values(fixes).filter(f => f.patched).length;
if (patchedCount === 0) {
    console.error('VERIFY_FAILED:No fixes were applied');
    process.exit(1);
}

// AST validation
try {
    acorn.parse(newCode, { ecmaVersion: 'latest', sourceType: 'script' });
} catch {
    try {
        acorn.parse(newCode, { ecmaVersion: 'latest', sourceType: 'module' });
    } catch (e) {
        console.error('VERIFY_FAILED:Post-patch AST parse failed: ' + e.message);
        process.exit(1);
    }
}

// Semantic validation
if (!newCode.includes(PATCH_MARKER)) {
    console.error('VERIFY_FAILED:Patch marker not found after rewrite');
    process.exit(1);
}

// Backup original file
const backupSuffix = process.env.BACKUP_SUFFIX || 'backup-refusal-no-switch';
const timestamp = new Date().toISOString().replace(/[:.]/g, '-').slice(0, 19);
const backupPath = cliPath + '.' + backupSuffix + '-' + timestamp;
fs.copyFileSync(cliPath, backupPath);
console.log('BACKUP:' + backupPath);

// Write patched file
fs.writeFileSync(cliPath, shebang + newCode);
console.log('SUCCESS:' + patchedCount);
'@

    # --------------------------------------------------------
    # Execute patch script
    # --------------------------------------------------------
    $tempPatchScript = Join-Path $env:TEMP "claude-fix-refusal-no-switch-$PID.js"
    $patchScript | Out-File -FilePath $tempPatchScript -Encoding UTF8

    # Set environment variable for backup suffix
    $env:BACKUP_SUFFIX = $BACKUP_SUFFIX

    $checkArg = if ($Check) { "--check" } else { "" }
    $output = & node $tempPatchScript $acornPath $cliPath $checkArg 2>&1
    $scriptExitCode = $LASTEXITCODE

    # Cleanup
    Remove-Item $tempPatchScript -ErrorAction SilentlyContinue

    # --------------------------------------------------------
    # Process output
    # --------------------------------------------------------
    foreach ($line in $output) {
        switch -Regex ($line) {
            "^ALREADY_PATCHED" { Write-Success "Already patched"; return 0 }
            "^PARSE_ERROR:(.+)" { Write-FixError "Failed to parse cli.js: $($Matches[1])"; return 1 }
            "^NOT_FOUND:(.+)" { Write-FixError "Target code not found: $($Matches[1])"; return 1 }
            "^FOUND:(.+)" { Write-Info "Found: $($Matches[1])" }
            "^PATCH:(.+)" { Write-Info "Patch: $($Matches[1])" }
            "^NEEDS_PATCH" {
                Write-Host ""
                Write-Warning "Patch needed - run without -Check to apply"
            }
            "^PATCH_COUNT:(.+)" {
                Write-Info "Need to patch $($Matches[1]) location(s)"
                return 1
            }
            "^BACKUP:(.+)" { Write-Host ""; Write-Host "Backup: $($Matches[1])" }
            "^SUCCESS:(.+)" {
                Write-Host ""
                Write-Success "Fix applied successfully! Patched $($Matches[1]) location(s)"
                Write-Host ""
                Write-Warning "Restart Claude Code for changes to take effect"
                Write-Host ""
                Write-Info "Set CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK=1 in ~/.claude/settings.json env to activate:"
                Write-Info "  Server fallback response accepted (turn continues)"
                Write-Info "  But session model stays unchanged (no persistent switch)"
            }
            "^VERIFY_FAILED:(.+)" { Write-FixError "Verification failed: $($Matches[1])"; return 1 }
        }
    }

    return $scriptExitCode
}

# ============================================================
# Entry point - Invoke main function with passed parameters
# ============================================================
Invoke-ClaudeCodeFix -Check:$Check -Restore:$Restore -Help:$Help -CliPath $CliPath
```



***macOS / Linux 系统***，新建 **`.sh`** 文件，复制以下代码执行

```
#!/bin/bash
#
# Claude Code Refusal Fallback No-Switch Fix Script
#
# THE BUG:
# CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK env var kills the entire refusal
# fallback mechanism — the turn stops dead on safety refusal, with no retry
# and no continuation. Users who don't want model switching have no way to
# keep the turn alive.
#
# ROOT CAUSE:
# ed() returns false when the env var is set, which short-circuits ALL
# fallback logic: no server-side fallback signalling, no client-side retry,
# no dialog prompt. The turn just dies on stop_reason "refusal".
#
# FIX POINTS:
# 1) ed() → always return true (fallback machinery stays active)
# 2) swapSession block → add env var guard so model is NOT persisted when set
#    The server's fallback response is still accepted (turn completes), but
#    mainLoopModel / mainLoopModelForSession / mainLoopModelOverride stay
#    unchanged — next turn uses the user's original model.
#
# Applicable: v2.1.170+ (requires swapSession/latch architecture)
#
# Usage:
#   ./apply-claude-code-refusal-no-switch-fix.sh                    # Apply fix (auto-detect)
#   ./apply-claude-code-refusal-no-switch-fix.sh /path/to/cli.js    # Apply fix to specific file
#   ./apply-claude-code-refusal-no-switch-fix.sh --check            # Check only
#   ./apply-claude-code-refusal-no-switch-fix.sh --restore          # Restore backup
#

set -e

# ============================================================
# Configuration - Modify these for each fix script
# ============================================================
BACKUP_SUFFIX="backup-refusal-no-switch"
FIX_DESCRIPTION="Change CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK to allow turn completion without model switch"

# ============================================================
# Color output functions
# ============================================================
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

success() { echo -e "${GREEN}[OK]${NC} $1"; }
warning() { echo -e "${YELLOW}[!]${NC} $1"; }
error() { echo -e "${RED}[X]${NC} $1"; }
info() { echo -e "${BLUE}[>]${NC} $1"; }

# ============================================================
# Argument parsing
# ============================================================
CHECK_ONLY=false
RESTORE=false
CLI_PATH_ARG=""

while [[ $# -gt 0 ]]; do
    case $1 in
        --check|-c) CHECK_ONLY=true; shift ;;
        --restore|-r) RESTORE=true; shift ;;
        --help|-h)
            echo "Usage: $0 [options] [cli.js path]"
            echo ""
            echo "$FIX_DESCRIPTION"
            echo ""
            echo "Arguments:"
            echo "  cli.js path    Path to cli.js file (optional, auto-detect if not provided)"
            echo ""
            echo "Options:"
            echo "  --check, -c    Check if fix is needed without making changes"
            echo "  --restore, -r  Restore original file from backup"
            echo "  --help, -h     Show help information"
            echo ""
            echo "Examples:"
            echo "  $0                                    # Auto-detect and apply fix"
            echo "  $0 /path/to/cli.js                    # Apply fix to specific file"
            echo "  $0 --check /path/to/cli.js            # Check specific file"
            echo "  $0 /path/to/cli.js --check            # Same as above"
            exit 0
            ;;
        -*)
            error "Unknown option: $1"
            exit 1
            ;;
        *)
            if [[ -z "$CLI_PATH_ARG" ]]; then
                CLI_PATH_ARG="$1"
            else
                error "Unexpected argument: $1"
                exit 1
            fi
            shift
            ;;
    esac
done

# ============================================================
# Find Claude Code cli.js path
# ============================================================
find_cli_path() {
    local locations=(
        "$HOME/.claude/local/node_modules/@cometix/claude-code/cli.js"
        "/usr/local/lib/node_modules/@cometix/claude-code/cli.js"
        "/usr/lib/node_modules/@cometix/claude-code/cli.js"
        "$HOME/.claude/local/node_modules/@anthropic-ai/claude-code/cli.js"
        "/usr/local/lib/node_modules/@anthropic-ai/claude-code/cli.js"
        "/usr/lib/node_modules/@anthropic-ai/claude-code/cli.js"
    )
    if command -v npm &> /dev/null; then
        local npm_root
        npm_root=$(npm root -g 2>/dev/null || true)
        if [[ -n "$npm_root" ]]; then
            locations+=("$npm_root/@cometix/claude-code/cli.js")
            locations+=("$npm_root/@anthropic-ai/claude-code/cli.js")
        fi
    fi
    for path in "${locations[@]}"; do
        if [[ -f "$path" ]]; then
            echo "$path"
            return 0
        fi
    done
    return 1
}

# ============================================================
# Determine CLI_PATH: use provided path or auto-detect
# ============================================================
if [[ -n "$CLI_PATH_ARG" ]]; then
    if [[ -f "$CLI_PATH_ARG" ]]; then
        CLI_PATH="$CLI_PATH_ARG"
        info "Using specified cli.js: $CLI_PATH"
    else
        error "Specified file not found: $CLI_PATH_ARG"
        exit 1
    fi
else
    CLI_PATH=$(find_cli_path) || {
        error "Claude Code cli.js not found"
        echo ""
        echo "Searched locations:"
        echo "  ~/.claude/local/node_modules/@cometix/claude-code/cli.js"
        echo "  /usr/local/lib/node_modules/@cometix/claude-code/cli.js"
        echo "  ~/.claude/local/node_modules/@anthropic-ai/claude-code/cli.js"
        echo "  /usr/local/lib/node_modules/@anthropic-ai/claude-code/cli.js"
        echo "  \$(npm root -g)/@cometix/claude-code/cli.js"
        echo "  \$(npm root -g)/@anthropic-ai/claude-code/cli.js"
        echo ""
        echo "Tip: You can specify the path directly:"
        echo "  $0 /path/to/cli.js"
        exit 1
    }
    info "Found Claude Code: $CLI_PATH"
fi

CLI_DIR=$(dirname "$CLI_PATH")

# ============================================================
# Restore backup
# ============================================================
if $RESTORE; then
    LATEST_BACKUP=$(ls -t "$CLI_DIR"/cli.js.${BACKUP_SUFFIX}-* 2>/dev/null | head -1)
    if [[ -n "$LATEST_BACKUP" ]]; then
        cp "$LATEST_BACKUP" "$CLI_PATH"
        success "Restored from backup: $LATEST_BACKUP"
        exit 0
    else
        error "No backup file found (cli.js.${BACKUP_SUFFIX}-*)"
        exit 1
    fi
fi

echo ""

# ============================================================
# Download acorn parser if needed
# ============================================================
ACORN_PATH="/tmp/acorn-claude-fix.js"
if [[ ! -f "$ACORN_PATH" ]]; then
    info "Downloading acorn parser..."
    curl -sL "https://unpkg.com/acorn@8.16.0/dist/acorn.js" -o "$ACORN_PATH" || {
        error "Failed to download acorn parser"
        exit 1
    }
fi

# ============================================================
# Node.js patch script (heredoc)
# ============================================================
PATCH_SCRIPT=$(mktemp)
cat > "$PATCH_SCRIPT" << 'PATCH_EOF'
const fs = require('fs');
const acornPath = process.argv[2];
const acorn = require(acornPath);

const cliPath = process.argv[3];
const checkOnly = process.argv[4] === '--check';
const backupSuffix = process.env.BACKUP_SUFFIX || 'backup-refusal-no-switch';

let code = fs.readFileSync(cliPath, 'utf-8');

// Preserve shebang
let shebang = '';
if (code.startsWith('#!')) {
    const idx = code.indexOf('\n');
    shebang = code.slice(0, idx + 1);
    code = code.slice(idx + 1);
}

// ============================================================
// Constants
// ============================================================
const ENV_VAR = 'CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK';
const PATCH_MARKER = 'CC_REFUSAL_NO_SWITCH_V1';

// Track fix status
let fixes = {
    edFunc: { found: false, patched: false, node: null },
    swapSession: { found: false, patched: false, node: null },
};

// Parse AST
let ast;
try {
    ast = acorn.parse(code, { ecmaVersion: "latest", sourceType: 'script' });
} catch (e) {
    try {
        ast = acorn.parse(code, { ecmaVersion: "latest", sourceType: 'module' });
    } catch (e2) {
        console.error('PARSE_ERROR:' + e2.message);
        process.exit(1);
    }
}

// AST helper: find nodes matching predicate
function findNodes(node, predicate, results = []) {
    if (!node || typeof node !== 'object') return results;
    if (predicate(node)) results.push(node);
    for (const key in node) {
        if (node[key] && typeof node[key] === 'object') {
            if (Array.isArray(node[key])) {
                node[key].forEach(child => findNodes(child, predicate, results));
            } else {
                findNodes(node[key], predicate, results);
            }
        }
    }
    return results;
}

// Get source code snippet from AST node
const src = (node) => code.slice(node.start, node.end);

// ============================================================
// Fix 1: Find ed() — the function that checks DISABLE_REFUSAL_FALLBACK
//
// Pattern A (v2.1.170):
//   function XX() { return !STORE.CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK }
//
// Pattern B (v2.1.162):
//   function XX() {
//     if (STORE.CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK) return !1;
//     return featureFlag(...)
//   }
// ============================================================

let edFunc = null;
let edStoreName = null;

const allFuncs = findNodes(ast, n =>
    (n.type === 'FunctionDeclaration' || n.type === 'FunctionExpression') &&
    n.params?.length === 0 &&
    n.body?.type === 'BlockStatement'
);

for (const fn of allFuncs) {
    const body = fn.body.body;

    // Pattern A: return !STORE.ENV_VAR
    if (body.length === 1 && body[0].type === 'ReturnStatement') {
        const arg = body[0].argument;
        if (arg?.type === 'UnaryExpression' && arg.operator === '!' &&
            arg.argument?.type === 'MemberExpression' &&
            arg.argument.property?.type === 'Identifier' &&
            arg.argument.property.name === ENV_VAR) {
            edFunc = fn;
            edStoreName = src(arg.argument.object);
            break;
        }
    }

    // Pattern B: if (STORE.ENV_VAR) return !1; return ...
    if (body.length === 2 && body[0].type === 'IfStatement' && body[1].type === 'ReturnStatement') {
        const test = body[0].test;
        if (test?.type === 'MemberExpression' &&
            test.property?.type === 'Identifier' &&
            test.property.name === ENV_VAR) {
            edFunc = fn;
            edStoreName = src(test.object);
            break;
        }
    }
}

if (edFunc) {
    const name = edFunc.id?.name || '<anon>';
    fixes.edFunc.found = true;
    fixes.edFunc.node = edFunc;
    console.log(`FOUND:edFunc "${name}" at byte ${edFunc.start} — checks ${ENV_VAR} on ${edStoreName}`);
}

// ============================================================
// Fix 2: Find swapSession block that persists model
//
// The if-test is a SequenceExpression (comma operator):
//   if (W=!0, S=o_.toModel, wLH(), BH=o_.toModel, i_.swapSession) { ... }
//
// The last expression in the sequence is the actual boolean test:
//   MemberExpression: *.swapSession
//
// We wrap it: *.swapSession && !STORE.ENV_VAR
// ============================================================

const ifStmts = findNodes(ast, n => n.type === 'IfStatement');

for (const node of ifStmts) {
    let test = node.test;

    // Unwrap SequenceExpression — the real test is the last expression
    let swapExpr = null;
    if (test?.type === 'SequenceExpression' && test.expressions?.length > 0) {
        const last = test.expressions[test.expressions.length - 1];
        if (last?.type === 'MemberExpression' &&
            last.property?.type === 'Identifier' &&
            last.property.name === 'swapSession') {
            swapExpr = last;
        }
    }
    // Also handle simple case: if (*.swapSession) { ... }
    if (!swapExpr && test?.type === 'MemberExpression' &&
        test.property?.type === 'Identifier' &&
        test.property.name === 'swapSession') {
        swapExpr = test;
    }

    if (!swapExpr) continue;

    const block = node.consequent;
    if (!block) continue;
    const blockSrc = src(block);

    // Verify this is the model persistence block
    if (blockSrc.includes('mainLoopModel') &&
        blockSrc.includes('.toModel') &&
        blockSrc.includes('mainLoopModelForSession')) {
        fixes.swapSession.found = true;
        fixes.swapSession.node = { ifNode: node, swapExpr: swapExpr };
        console.log(`FOUND:swapSession block at byte ${node.start} (swap test at ${swapExpr.start})`);
        break;
    }
}

// ============================================================
// Check if already patched or patterns not found
// ============================================================

const allFound = Object.values(fixes).some(f => f.found);
if (!allFound) {
    const alreadyPatched = code.includes(PATCH_MARKER);
    if (alreadyPatched) {
        console.log('ALREADY_PATCHED');
        process.exit(2);
    }

    // Provide specific diagnostics
    if (!code.includes(ENV_VAR)) {
        console.error('NOT_FOUND:CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK not found in code (pre-v2.1.162?)');
    } else if (!code.includes('swapSession')) {
        console.error('NOT_FOUND:swapSession architecture not found (pre-v2.1.170?)');
    } else {
        console.error('NOT_FOUND:Unable to locate ed() function or swapSession block');
    }
    process.exit(1);
}

if (checkOnly) {
    console.log('NEEDS_PATCH');
    const count = Object.values(fixes).filter(f => f.found).length;
    console.log('PATCH_COUNT:' + count);
    process.exit(1);
}

// ============================================================
// Apply fixes using AST node positions
// ============================================================

let newCode = code;

function replaceAt(str, start, end, replacement) {
    return str.slice(0, start) + replacement + str.slice(end);
}

let replacements = [];

// Fix 1: ed() always returns true
if (fixes.edFunc.found && fixes.edFunc.node) {
    const node = fixes.edFunc.node;
    replacements.push({
        start: node.body.start,
        end: node.body.end,
        replacement: `{return!0/*${PATCH_MARKER}*/}`,
        name: 'edFunc'
    });
    fixes.edFunc.patched = true;
    console.log('PATCH:edFunc - ed() now always returns true (fallback machinery stays active)');
}

// Fix 2: swapSession guard — wrap the swapSession MemberExpression
if (fixes.swapSession.found && fixes.swapSession.node) {
    const { swapExpr } = fixes.swapSession.node;
    const swapSrc = src(swapExpr);
    const store = edStoreName || '$_';
    replacements.push({
        start: swapExpr.start,
        end: swapExpr.end,
        replacement: `${swapSrc}&&!${store}.${ENV_VAR}`,
        name: 'swapSession'
    });
    fixes.swapSession.patched = true;
    console.log(`PATCH:swapSession - model persistence guarded by !${store}.${ENV_VAR}`);
}

// Apply AST-based replacements from end to start to preserve positions
replacements.sort((a, b) => b.start - a.start);
for (const r of replacements) {
    newCode = replaceAt(newCode, r.start, r.end, r.replacement);
}

// ============================================================
// Verify and save
// ============================================================

const patchedCount = Object.values(fixes).filter(f => f.patched).length;
if (patchedCount === 0) {
    console.error('VERIFY_FAILED:No fixes were applied');
    process.exit(1);
}

// AST validation
try {
    acorn.parse(newCode, { ecmaVersion: 'latest', sourceType: 'script' });
} catch {
    try {
        acorn.parse(newCode, { ecmaVersion: 'latest', sourceType: 'module' });
    } catch (e) {
        console.error('VERIFY_FAILED:Post-patch AST parse failed: ' + e.message);
        process.exit(1);
    }
}

// Semantic validation
if (!newCode.includes(PATCH_MARKER)) {
    console.error('VERIFY_FAILED:Patch marker not found after rewrite');
    process.exit(1);
}

// Backup original file
const timestamp = new Date().toISOString().replace(/[:.]/g, '-').slice(0, 19);
const backupPath = cliPath + '.' + backupSuffix + '-' + timestamp;
fs.copyFileSync(cliPath, backupPath);
console.log('BACKUP:' + backupPath);

// Write patched file
fs.writeFileSync(cliPath, shebang + newCode);
console.log('SUCCESS:' + patchedCount);
PATCH_EOF

# ============================================================
# Execute patch script
# ============================================================
CHECK_ARG=""
if $CHECK_ONLY; then
    CHECK_ARG="--check"
fi

export BACKUP_SUFFIX
OUTPUT=$(node "$PATCH_SCRIPT" "$ACORN_PATH" "$CLI_PATH" "$CHECK_ARG" 2>&1) || true
EXIT_CODE=$?

rm -f "$PATCH_SCRIPT"

# ============================================================
# Process output
# ============================================================
while IFS= read -r line; do
    case "$line" in
        ALREADY_PATCHED)
            success "Already patched"
            exit 0
            ;;
        PARSE_ERROR:*)
            error "Failed to parse cli.js: ${line#PARSE_ERROR:}"
            exit 1
            ;;
        NOT_FOUND:*)
            error "Target code not found: ${line#NOT_FOUND:}"
            exit 1
            ;;
        FOUND:*)
            info "Found: ${line#FOUND:}"
            ;;
        PATCH:*)
            info "Patch: ${line#PATCH:}"
            ;;
        NEEDS_PATCH)
            echo ""
            warning "Patch needed - run without --check to apply"
            ;;
        PATCH_COUNT:*)
            info "Need to patch ${line#PATCH_COUNT:} location(s)"
            exit 1
            ;;
        BACKUP:*)
            echo ""
            echo "Backup: ${line#BACKUP:}"
            ;;
        SUCCESS:*)
            echo ""
            success "Fix applied successfully! Patched ${line#SUCCESS:} location(s)"
            echo ""
            warning "Restart Claude Code for changes to take effect"
            echo ""
            info "Set CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK=1 in ~/.claude/settings.json env to activate:"
            info "  Server fallback response accepted (turn continues)"
            info "  But session model stays unchanged (no persistent switch)"
            ;;
        VERIFY_FAILED:*)
            error "Verification failed: ${line#VERIFY_FAILED:}"
            exit 1
            ;;
    esac
done <<< "$OUTPUT"

exit $EXIT_CODE
```
