(function(){
if (window.__humanTyperInstalled) return;
window.__humanTyperInstalled = true;


let settings = { minDelay: 20, maxDelay: 120, variance: 0.35 };
chrome.storage.sync.get(["minDelay","maxDelay","variance"], (s) => {
if (!chrome.runtime.lastError) {
if (typeof s.minDelay === 'number') settings.minDelay = s.minDelay;
if (typeof s.maxDelay === 'number') settings.maxDelay = s.maxDelay;
if (typeof s.variance === 'number') settings.variance = s.variance;
}
});


chrome.runtime.onMessage.addListener(async (msg, sender, sendResponse) => {
if (msg.type === 'typeText' && msg.text) {
await typeText(msg.text);
sendResponse({done:true});
}
});


function randomDelay() {
const base = settings.minDelay + Math.random() * (settings.maxDelay - settings.minDelay);
const jitter = 1 + (Math.random() * 2 - 1) * settings.variance;
return Math.max(1, Math.round(base * jitter));
}


async function typeText(text) {
for (let i = 0; i < text.length; i++) {
const ch = text[i];
let success = false;
try {
success = document.execCommand('insertText', false, ch);
} catch (e) {}


if (!success) {
const sel = window.getSelection && window.getSelection();
if (sel && sel.rangeCount) {
const range = sel.getRangeAt(0);
range.deleteContents();
const node = document.createTextNode(ch);
range.insertNode(node);
range.setStartAfter(node);
range.setEndAfter(node);
sel.removeAllRanges();
sel.addRange(range);
})();
