# F1.7: "Free Up Space" Utility - Architecture Summary

This is a **client-side (React Native) feature** that does not have its own dedicated server architecture. It reuses an existing API.

The app's logic is as follows:
1.  **Fetch Cloud List:** The app calls our `F1.3: Main Gallery View` API to get a definitive list of all successfully backed-up photos.
2.  **Fetch Local List:** The app scans the phone's local camera roll.
3.  **Compare:** It compares the two lists to find all photos that exist in *both* places.
4.  **Execute:** After user confirmation, the app loops through this list and deletes *only the local copies* from the user's device.