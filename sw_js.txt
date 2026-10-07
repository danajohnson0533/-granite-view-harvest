// GVF Harvest service worker — background dumping notifications, version 1.6.17.
const CACHE='gvf-harvest-shell-v1-6-17';
self.addEventListener('install',event=>{event.waitUntil(self.skipWaiting())});
self.addEventListener('activate',event=>{event.waitUntil(self.clients.claim())});
self.addEventListener('fetch',event=>{
  if(event.request.method!=='GET'||new URL(event.request.url).origin!==self.location.origin)return;
  event.respondWith(fetch(event.request).then(response=>{
    if(response.ok){const copy=response.clone();event.waitUntil(caches.open(CACHE).then(c=>c.put(event.request,copy)).catch(()=>{}))}
    return response;
  }).catch(async()=>await caches.match(event.request)||Response.error()));
});
self.addEventListener('push',event=>{
 let data={};try{data=event.data?event.data.json():{}}catch{data={body:event.data?event.data.text():''}}
 const title=String(data.title||'GVF Harvest');
 const options={body:String(data.body||'Truck is dumping'),tag:String(data.tag||'gvf-dumping'),renotify:true,data:{url:String(data.url||'./')}};
 event.waitUntil(self.registration.showNotification(title,options));
});
self.addEventListener('notificationclick',event=>{
 event.notification.close();
 event.waitUntil((async()=>{
   const wanted=new URL('./',self.registration.scope).href;
   const windows=await clients.matchAll({type:'window',includeUncontrolled:true});
   for(const w of windows){if(w.url.startsWith(wanted)){await w.focus();return}}
   if(clients.openWindow)await clients.openWindow(wanted);
 })());
});
