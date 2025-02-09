<script>
(function() {
  // Get cookie consent
  function getCookieConsent() {
    const consent = document.cookie.split('; ').find(row => row.startsWith('cookie_consent='));
    return consent ? consent.split('=')[1] : null;
  }

  // Retrieve user information from localStorage
  let user_age = localStorage.getItem('user_age') || "unknown";
  let user_gender = localStorage.getItem('user_gender') || "unknown";
  let user_region = localStorage.getItem('user_region') || "unknown";
  let user_interest = localStorage.getItem('user_interest') || "unknown";

  // Generate session ID
  function generateSessionId() {
    return 'sess_' + Math.random().toString(36).substr(2, 9);
  }

  let session_id = localStorage.getItem('tracking_session_id');
  if (!session_id) {
    session_id = generateSessionId();
    localStorage.setItem('tracking_session_id', session_id);
  }

  // Additional data if consent is given
  const consent = getCookieConsent();
  let additionalData = {};

  if (consent === 'true') {
    additionalData = {
      age: user_age,
      gender: user_gender,
      region: user_region,
      interest: user_interest
    };
  } else {
    additionalData = {
      age: "unknown",
      gender: "unknown",
      region: "unknown",
      interest: "unknown"
    };
  }

  // Send tracking event
  function sendTrackingEvent(eventType, additionalData = {}) {
    fetch("https://app.we-eva.com/version-test/api/1.1/wf/event-tracking", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        tracking_id: "SQNQQWSZCI",  // Tracking ID
        session_id: session_id,
        event_type: eventType,
        url: window.location.href,
        timestamp: Date.now(),
        user_agent: navigator.userAgent,
        screen_width: window.screen.width,
        screen_height: window.screen.height,
        additional_data: JSON.stringify(additionalData)
      })
    })
    .then(response => response.json())
    .then(data => console.log("Tracking Data:", data))
    .catch(error => console.error("Tracking Error:", error));
  }

  // Send page view event
  sendTrackingEvent("page_view", additionalData);

  // Track page transition
  let startTime = Date.now();
  let totalDuration = 0;

  function trackPageTransition() {
    let duration = Date.now() - startTime + totalDuration;
    sendTrackingEvent("page_transition", { duration: duration });
    totalDuration = duration;
    startTime = Date.now(); 
  }

  // Track page exit event
  window.addEventListener("beforeunload", function() {
    let duration = Date.now() - startTime + totalDuration;
    sendTrackingEvent("page_exit", { duration: duration });
    totalDuration = duration;
    startTime = Date.now();
  });

  // Track page transitions via popstate
  window.addEventListener("popstate", function() {
    if (window.location.href !== location.href) {
      trackPageTransition(); 
    }
  });

  // Track hash changes
  window.addEventListener("hashchange", function() {
    trackPageTransition(); 
  });

})();
</script>
