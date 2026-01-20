

---

### Building the SEPSIS-BAYES Model: From Concept to Working System

The Sepsis-Bayes model is built around a single idea: at any moment in time, a patient has a probability of being septic, and that probability should be updated whenever new information becomes available. The system therefore does not wait for a “complete case.” It reasons continuously.

The model is composed of three logical layers: prior estimation, evidence likelihood estimation, and Bayesian updating. These layers interact in real time.

---

### Step 1: Defining the Patient State

At the heart of the system is a patient state object. Conceptually, this is a record that stores what the system currently believes about the patient.

For each patient, the system stores:
• the current probability of sepsis
• the prior probability used at admission
• a log of observed evidence and timestamps
• the most recent posterior probability

This patient state updates every time a new observation arrives. Importantly, the probability is never reset unless the patient is discharged or definitively diagnosed.

---

### Step 2: Computing the Prior Programmatically

The prior probability is computed at admission or first contact. Instead of being guessed, it is calculated using weighted contextual factors. In the first version of the model, these factors are deliberately simple and interpretable.

The prior is computed as a baseline hospital prevalence, adjusted by risk modifiers.

For example, consider a baseline hospital-wide sepsis prevalence of 6 percent at a regional referral hospital such as Mbale. This value can be estimated from retrospective admissions data.

We then adjust this baseline using multiplicative or additive factors. A patient admitted to the ICU might multiply the baseline by 2.5. Recent surgery might multiply it by 1.6. Known infection could multiply it by 1.8. Advanced age might multiply it by 1.3.

So if a 65-year-old post-surgical patient with suspected pneumonia is admitted to ICU, the prior becomes:

P(S) = 0.06 × 2.5 × 1.6 × 1.8 × 1.3 ≈ 0.56

That looks high, and that is intentional — such a patient is genuinely high risk. For a younger patient in a general ward with no known infection, the same hospital baseline might remain close to 4–6 percent.

In code, this is simply a function that takes patient metadata and outputs a number between 0 and 1. The key is that the logic is transparent and editable by clinicians and researchers.

---

### Step 3: Defining Evidence Likelihoods

Once the prior exists, the system waits for evidence. Each piece of evidence is treated as a probabilistic signal.

For each signal, we define two values:
• the probability of observing the signal if the patient truly has sepsis
• the probability of observing the signal if the patient does not have sepsis

These values are stored in a likelihood table.

For instance, elevated lactate might be observed in about 70 percent of septic patients but only 20 percent of non-septic patients. Fever might be present in 65 percent of sepsis cases but also in 40 percent of non-sepsis cases. Hypotension might appear in 55 percent of sepsis cases and only 15 percent of non-sepsis cases.

These numbers do not need to be perfect at first. They can be sourced from published studies, regional audits, or even conservative estimates agreed upon by clinicians. The Bayesian framework is robust to imperfect assumptions, as long as they are reasonable.

---

### Step 4: The Bayesian Update Engine (The Core Code Logic)

Now we reach the spine of the system.

Every time a new piece of evidence arrives, the system recalculates the posterior probability using Bayes’ rule. Conceptually, this happens as follows:

The current probability of sepsis is multiplied by the likelihood of the new evidence given sepsis. The probability of non-sepsis is multiplied by the likelihood of the same evidence given no sepsis. These two values are then normalized so that they sum to one.

This update happens sequentially. The output of one update becomes the input prior for the next.

Mathematically, if the current belief is P(S), and a new observation E arrives, the update is:

P(S | E) = [P(E | S) × P(S)] / [P(E | S) × P(S) + P(E | ¬S) × (1 − P(S))]

This single equation is called repeatedly. It is computationally cheap, which makes it suitable even for low-resource hospital systems.

---

### Step 5: Real-Time Example Walkthrough

Consider a real-time case at Mulago National Referral Hospital.

A patient is admitted to the emergency department with suspected infection. Based on hospital location, age, and known infection, the system assigns a prior probability of 0.18.

Two hours later, the nurse records a fever. Fever has a likelihood of 0.65 if septic and 0.40 if not septic. The probability updates to approximately 0.26.

An hour later, the patient’s heart rate rises significantly. This observation shifts the probability to around 0.37.

A lactate result comes back elevated. Because elevated lactate strongly discriminates sepsis, the probability jumps to over 0.60.

At this point, even if blood cultures are still pending, the system flags the patient as high risk and recommends immediate sepsis protocol activation. Importantly, the system does not claim certainty. It claims urgency, and it explains why.

---

### Step 6: Decision Thresholds and Clinical Use

The Bayesian engine itself does not make decisions. It produces probabilities. Decisions are made by comparing those probabilities against thresholds defined by hospital policy.

For example, a probability above 30 percent might prompt closer monitoring. Above 50 percent might trigger early antibiotics. Above 70 percent might initiate full sepsis bundle care.

These thresholds can differ between hospitals. A district hospital with limited ICU capacity may act earlier, while a national referral hospital may tolerate more uncertainty before escalation.

---

### Step 7: How This Becomes a Real Project

From a student or research perspective, this model can be built incrementally.

The first milestone is a simulation using retrospective patient data. The model is run “in shadow mode,” meaning it computes probabilities without influencing care. These probabilities are then compared to actual outcomes to evaluate how early the system would have flagged sepsis.

The second milestone is calibration. Likelihood values and priors are adjusted to improve accuracy using local data from hospitals such as Mbarara or Mbale.

The final milestone is a prototype dashboard that shows clinicians a probability curve over time, along with explanations for each change.

---

### The Big Picture Insight

What you are building here is not just a sepsis detector. You are building a **reasoning system under uncertainty**, something that medicine has needed for decades but rarely implemented explicitly.

Bayes does not replace clinical judgment. It makes clinical judgment visible, measurable, and improvable.

---
