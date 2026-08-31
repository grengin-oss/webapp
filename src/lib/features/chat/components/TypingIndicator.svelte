<!--
SPDX-FileCopyrightText: 2026 Perter Technology Solutions Private Limited
SPDX-License-Identifier: Apache-2.0
-->

<script lang="ts">
  import { _ } from 'svelte-i18n';

  interface Props {
    /**
     * True while an image-generation model is working. Image models can run for
     * a long time with no text deltas, so the design replaces the three dots
     * with a 280px placeholder card carrying a spinning gradient ring
     * (chat-empty-state.html, .ai-generation-loading).
     */
    generatingImage?: boolean;
  }

  let { generatingImage = false }: Props = $props();
</script>

{#if generatingImage}
  <div
    class="typing-indicator"
    role="status"
    aria-label={$_('chat.message.creatingImage')}
    aria-live="polite"
  >
    <div class="ai-generation-loading">
      <div class="orbital-rings" aria-hidden="true">
        <div class="orbital-ring"><div class="orbital-ring-ellipse"></div></div>
        <div class="orbital-ring"><div class="orbital-ring-ellipse"></div></div>
        <div class="orbital-ring"><div class="orbital-ring-ellipse"></div></div>
      </div>
      <span class="loading-status">{$_('chat.message.creatingImage')}</span>
    </div>
  </div>
{:else}
  <div class="typing-indicator" role="status" aria-label={$_('components.typingIndicator')} aria-live="polite">
    <div class="typing-content">
      <div class="typing-dots" aria-hidden="true">
        <span class="dot"></span>
        <span class="dot"></span>
        <span class="dot"></span>
      </div>
    </div>
  </div>
{/if}

<style>
  .typing-indicator {
    display: flex;
    align-items: flex-start;
    margin-bottom: var(--space-xl);
    animation: slideIn 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }

  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .typing-content {
    max-width: 80%;
    width: fit-content;
    background: color-mix(in oklab, var(--glass-bg-dark) 60%, var(--btn-secondary));
    backdrop-filter: blur(0.625rem);
    -webkit-backdrop-filter: blur(0.625rem);
    padding: var(--space-lg) var(--space-xl);
    border-radius: var(--radius-lg);
    box-shadow: var(--glass-edge-glow), var(--glass-shadow-light);
  }

  .typing-dots {
    display: flex;
    align-items: center;
    gap: var(--space-sm);
  }

  .dot {
    width: 8px;
    height: 8px;
    background: var(--text-secondary);
    border-radius: 50%;
    animation: bounce 1.4s infinite ease-in-out;
  }

  .dot:nth-child(1) {
    animation-delay: -0.32s;
  }

  .dot:nth-child(2) {
    animation-delay: -0.16s;
  }

  @keyframes bounce {
    0%, 80%, 100% {
      transform: scale(0.8);
      opacity: 0.5;
    }
    40% {
      transform: scale(1.2);
      opacity: 1;
    }
  }

  /* ---- image generation placeholder (design: .ai-generation-loading) ---- */
  .ai-generation-loading {
    width: 280px;
    height: 280px;
    max-width: 100%;
    border-radius: 16px;
    background: var(--gx-ring-soft);
    display: flex;
    flex-direction: column;
    gap: 20px;
    justify-content: center;
    align-items: center;
    padding: 24px;
    box-sizing: border-box;
  }

  /* Three orbiting ellipses (design: .orbital-rings). Each ring is a 100x100
     box holding one flattened 100x56 ellipse pinned at top:22px so it orbits
     around the box centre; the ellipses are pre-rotated 0/60/120deg and the
     middle one spins the other way, which is what reads as a sphere. */
  .orbital-rings {
    position: relative;
    width: 120px;
    height: 120px;
    flex-shrink: 0;
  }

  .orbital-ring {
    position: absolute;
    left: 10px;
    top: 10px;
    width: 100px;
    height: 100px;
  }

  .orbital-ring-ellipse {
    position: absolute;
    left: 0;
    top: 22px;
    width: 100px;
    height: 56px;
    opacity: 0.8;
    border-radius: 50%;
    background: rgba(0, 0, 0, 0);
    border: 2px solid;
  }

  .orbital-ring:nth-child(1) {
    animation: ring-spin 2.4s linear infinite;
  }

  .orbital-ring:nth-child(1) .orbital-ring-ellipse {
    border-color: rgb(79, 114, 210);
    transform: rotate(0deg);
  }

  .orbital-ring:nth-child(2) {
    animation: ring-spin-rev 2.4s linear infinite;
  }

  .orbital-ring:nth-child(2) .orbital-ring-ellipse {
    border-color: rgb(76, 169, 122);
    transform: rotate(60deg);
  }

  .orbital-ring:nth-child(3) {
    animation: ring-spin 2.4s linear infinite;
  }

  .orbital-ring:nth-child(3) .orbital-ring-ellipse {
    border-color: rgb(147, 174, 234);
    transform: rotate(120deg);
  }

  @keyframes ring-spin {
    to {
      transform: rotate(360deg);
    }
  }

  @keyframes ring-spin-rev {
    to {
      transform: rotate(-360deg);
    }
  }

  @media (prefers-reduced-motion: reduce) {
    .orbital-ring {
      animation-duration: 12s;
    }
  }

  .loading-status {
    font-family: var(--gx-font);
    font-size: 13px;
    color: var(--gx-slate-400);
  }

  @media (max-width: 768px) {
    .typing-content {
      max-width: 90%;
      padding: var(--space-md) var(--space-lg);
    }

    .ai-generation-loading {
      width: 240px;
      height: 240px;
    }
  }
</style>
