<script>
    import { onMount } from 'svelte';
    
    let jobDescription = $state('');
    let currentStep = $state('input'); // 'input', 'chat', 'details', 'confirmation', 'bids', 'contract', 'payment'
    let chatMessages = $state([]);
    let userEmail = $state('');
    let userName = $state('');
    let userPhone = $state('');
    let isLoading = $state(false);
    
    let jobId = $state('');
    let bids = $state([]);
    let selectedBid = $state(null);
    let contractSigned = false;
    let paymentAmount = $state(0);
    let paymentMethod = $state('card');
    
    // Sample AI questions based on job type
    const getAIQuestions = (description) => {
      const lowerDesc = description.toLowerCase();
      
      if (lowerDesc.includes('kitchen') || lowerDesc.includes('install')) {
        return [
          "What type of kitchen installation do you need? (cabinets, appliances, countertops, etc.)",
          "What's the approximate size of your kitchen?",
          "Do you have the materials already, or do you need the contractor to provide them?",
          "What's your preferred timeline for completion?",
          "What's your budget range for this project?"
        ];
      } else if (lowerDesc.includes('carpentry') || lowerDesc.includes('wood')) {
        return [
          "What type of carpentry work do you need? (furniture, repairs, custom build, etc.)",
          "What are the dimensions or scope of the project?",
          "Do you have specific materials in mind?",
          "Where is this work being done? (indoor/outdoor)",
          "What's your target completion date?"
        ];
      } else {
        return [
          "Can you provide more details about what exactly needs to be done?",
          "What's the scope or size of this project?",
          "Do you have any specific requirements or preferences?",
          "What's your preferred timeline?",
          "What's your budget range?"
        ];
      }
    };
    
    let currentQuestions = $state([]);
    let currentQuestionIndex = $state(0);
    let answers = [];
    
    const startJobPosting = () => {
      if (!jobDescription.trim()) return;
      
      currentQuestions = getAIQuestions(jobDescription);
      currentStep = 'chat';
      chatMessages = [
        {
          type: 'ai',
          content: `Great! I'll help you create a detailed job posting for: "${jobDescription}". Let me ask you a few questions to make sure contractors have all the information they need.`
        },
        {
          type: 'ai',
          content: currentQuestions[0]
        }
      ];
    };
    
    const handleChatResponse = (answer) => {
      if (!answer.trim()) return;
      
      // Add user's answer to chat
      chatMessages = [...chatMessages, { type: 'user', content: answer }];
      answers = [...answers, answer];
      
      currentQuestionIndex++;
      
      if (currentQuestionIndex < currentQuestions.length) {
        // Ask next question
        setTimeout(() => {
          chatMessages = [...chatMessages, { 
            type: 'ai', 
            content: currentQuestions[currentQuestionIndex] 
          }];
        }, 1000);
      } else {
        // All questions answered, move to details collection
        setTimeout(() => {
          chatMessages = [...chatMessages, { 
            type: 'ai', 
            content: "Perfect! Now I just need your contact information to post your job and connect you with qualified contractors." 
          }];
          currentStep = 'details';
        }, 1000);
      }
    };
    
    const submitJobPosting = async () => {
      if (!userName.trim() || !userEmail.trim()) return;
      
      isLoading = true;
      
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      jobId = 'JOB-' + Math.random().toString(36).substr(2, 9).toUpperCase();
      
      // Simulate bids coming in
      setTimeout(() => {
        generateSampleBids();
        currentStep = 'bids';
      }, 3000);
      
      currentStep = 'confirmation';
      isLoading = false;
    };
    
    const generateSampleBids = () => {
      const contractors = [
        {
          id: 1,
          name: "Mike's Construction",
          rating: 4.9,
          reviews: 127,
          price: Math.floor(Math.random() * 500) + 200,
          timeline: "2-3 days",
          description: "Experienced in kitchen installations with 15+ years in the business. I provide all materials and guarantee quality work.",
          avatar: "https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=150&h=150&fit=crop&crop=face"
        },
        {
          id: 2,
          name: "Sarah Johnson Carpentry",
          rating: 4.8,
          reviews: 89,
          price: Math.floor(Math.random() * 400) + 250,
          timeline: "1-2 days",
          description: "Specialized in custom carpentry work. Licensed, insured, and committed to exceeding expectations.",
          avatar: "https://images.unsplash.com/photo-1494790108755-2616b612b786?w=150&h=150&fit=crop&crop=face"
        },
        {
          id: 3,
          name: "Pro Home Services",
          rating: 4.7,
          reviews: 203,
          price: Math.floor(Math.random() * 600) + 180,
          timeline: "3-4 days",
          description: "Full-service home improvement company. We handle everything from permits to cleanup.",
          avatar: "https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?w=150&h=150&fit=crop&crop=face"
        }
      ];
      
      bids = contractors.sort((a, b) => a.price - b.price);
    };
    
    const selectBid = (bid) => {
      selectedBid = bid;
      paymentAmount = bid.price;
      currentStep = 'contract';
    };
    
    const signContract = async () => {
      isLoading = true;
      
      // Simulate contract signing
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      contractSigned = true;
      currentStep = 'payment';
      isLoading = false;
    };
    
    const processPayment = async () => {
      isLoading = true;
      
      // Simulate payment processing
      await new Promise(resolve => setTimeout(resolve, 3000));
      
      isLoading = false;
      
      // Show success and reset
      alert('Payment processed successfully! Your contractor will be notified and work will begin as scheduled.');
      resetForm();
    };
    
    const resetForm = () => {
      jobDescription = '';
      currentStep = 'input';
      chatMessages = [];
      userEmail = '';
      userName = '';
      userPhone = '';
      currentQuestionIndex = 0;
      answers = [];
      isLoading = false;
      jobId = '';
      bids = [];
      selectedBid = null;
      contractSigned = false;
      paymentAmount = 0;
    };
    
    let chatInput = $state('');
    
    const handleChatSubmit = (e) => {
      e.preventDefault();
      if (chatInput.trim()) {
        handleChatResponse(chatInput);
        chatInput = '';
      }
    };
  </script>
  
  <main class="min-h-screen bg-background grid-pattern">
    <header class="border-b border-border/50">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between items-center h-16">
          <div class="flex items-center space-x-4">
            <div class="text-2xl font-bold text-foreground">QuickHire</div>
            <div class="hidden sm:block text-sm text-muted-foreground">
              Post small jobs. Get instant bids.
            </div>
          </div>
          <button class="px-4 py-2 text-sm bg-primary text-primary-foreground rounded-lg hover:bg-primary/90 transition-colors">
            Sign In
          </button>
        </div>
      </div>
    </header>
  
    <div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
      {#if currentStep === 'input'}
        <div class="text-center mb-16 fade-in">
          <h1 class="text-4xl sm:text-6xl font-bold text-foreground mb-6 text-balance font-serif">
            The fastest way to post
            <span class="text-muted-foreground">small jobs</span>
          </h1>
          <p class="text-xl text-muted-foreground mb-12 text-pretty max-w-2xl mx-auto font-medium">
            Simply describe what you need help with. Our AI will ask clarifying questions, 
            then connect you with qualified contractors instantly.
          </p>
          
          <div class="max-w-2xl mx-auto">
            <form onsubmit={startJobPosting} class="relative flex flex-row items-center">
              <label for="jobDescription" class="sr-only">Job Description</label>
              <input
                id="jobDescription"
                bind:value={jobDescription}
                type="text"
                placeholder="What do you need help with? (e.g., install kitchen cabinets, fix deck railing...)"
                class="w-full px-6 py-4 pr-24 text-lg bg-input border border-border rounded-xl text-foreground placeholder-muted-foreground focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent shadow-sm"
              />
              <button
                type="submit"
                disabled={!jobDescription.trim()}
                class="absolute right-2  px-4 py-2 bg-primary text-primary-foreground rounded-lg hover:bg-primary/90 disabled:opacity-50 disabled:cursor-not-allowed transition-all font-medium text-sm whitespace-nowrap"
              >
                Start →
              </button>
            </form>
          </div>
        </div>
  
        <div class="grid md:grid-cols-3 gap-8 mb-16">
          <div class="glass rounded-xl p-6 fade-in shadow-sm">
            <div class="w-12 h-12 bg-primary/10 rounded-lg flex items-center justify-center mb-4">
              <svg class="w-6 h-6 text-primary" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
              </svg>
            </div>
            <h3 class="text-xl font-semibold text-foreground mb-2">AI-Powered Matching</h3>
            <p class="text-muted-foreground">Our AI asks the right questions to match you with perfect contractors for your specific needs.</p>
          </div>
  
          <div class="glass rounded-xl p-6 fade-in shadow-sm" style="animation-delay: 0.1s">
            <div class="w-12 h-12 bg-primary/10 rounded-lg flex items-center justify-center mb-4">
              <svg class="w-6 h-6 text-primary" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
              </svg>
            </div>
            <h3 class="text-xl font-semibold text-foreground mb-2">Secure Payments</h3>
            <p class="text-muted-foreground">Built-in contract signing and secure payment processing. Money held in escrow until job completion.</p>
          </div>
  
          <div class="glass rounded-xl p-6 fade-in shadow-sm" style="animation-delay: 0.2s">
            <div class="w-12 h-12 bg-primary/10 rounded-lg flex items-center justify-center mb-4">
              <svg class="w-6 h-6 text-primary" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 3v4M3 5h4M6 17v4m-2-2h4m5-16l2.286 6.857L21 12l-5.714 2.143L13 21l-2.286-6.857L5 12l5.714-2.143L13 3z" />
              </svg>
            </div>
            <h3 class="text-xl font-semibold text-foreground mb-2">Instant Bids</h3>
            <p class="text-muted-foreground">Get competitive bids from verified contractors within minutes, not days.</p>
          </div>
        </div>
      {:else if currentStep === 'chat'}
        <div class="max-w-2xl mx-auto fade-in">
          <div class="bg-card border border-border rounded-xl p-6 mb-6">
            <div class="flex items-center mb-4">
              <div class="w-8 h-8 bg-primary rounded-full flex items-center justify-center mr-3">
                <svg class="w-4 h-4 text-primary-foreground" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z" />
                </svg>
              </div>
              <div>
                <h3 class="font-semibold text-foreground">AI Assistant</h3>
                <p class="text-sm text-muted-foreground">Helping you create the perfect job posting</p>
              </div>
            </div>
            
            <div class="space-y-4 mb-6 max-h-96 overflow-y-auto">
              {#each chatMessages as message}
                <div class="flex {message.type === 'user' ? 'justify-end' : 'justify-start'}">
                  <div class="max-w-xs lg:max-w-md px-4 py-2 rounded-lg {message.type === 'user' ? 'bg-primary text-primary-foreground' : 'bg-muted text-muted-foreground'}">
                    {message.content}
                  </div>
                </div>
              {/each}
            </div>
            
            {#if currentQuestionIndex < currentQuestions.length}
              <form onsubmit={handleChatSubmit} class="flex gap-2">
                <input
                  id="chatInput"
                  bind:value={chatInput}
                  type="text"
                  placeholder="Type your answer..."
                  class="flex-1 px-4 py-2 bg-input border border-border rounded-lg text-foreground placeholder-muted-foreground focus:outline-none focus:ring-2 focus:ring-ring"
                />
                <button
                  type="submit"
                  disabled={!chatInput.trim()}
                  class="px-4 py-2 bg-primary text-primary-foreground rounded-lg hover:bg-primary/90 disabled:opacity-50 transition-colors"
                >
                  Send
                </button>
              </form>
            {/if}
          </div>
        </div>
      {:else if currentStep === 'details'}
        <div class="max-w-md mx-auto fade-in">
          <div class="bg-card border border-border rounded-xl p-6">
            <h3 class="text-2xl font-bold text-foreground mb-6 text-center">Almost Done!</h3>
            
            <form onsubmit={submitJobPosting} class="space-y-4">
              <div>
                <label for="userName" class="block text-sm font-medium text-foreground mb-2">Full Name</label>
                <input
                  id="userName"
                  bind:value={userName}
                  type="text"
                  required
                  class="w-full px-4 py-2 bg-input border border-border rounded-lg text-foreground focus:outline-none focus:ring-2 focus:ring-ring"
                />
              </div>
              
              <div>
                <label for="userEmail" class="block text-sm font-medium text-foreground mb-2">Email Address</label>
                <input
                  id="userEmail"
                  bind:value={userEmail}
                  type="email"
                  required
                  class="w-full px-4 py-2 bg-input border border-border rounded-lg text-foreground focus:outline-none focus:ring-2 focus:ring-ring"
                />
              </div>
              
              <div>
                <label for="userPhone" class="block text-sm font-medium text-foreground mb-2">Phone Number (Optional)</label>
                <input
                  id="userPhone"
                  bind:value={userPhone}
                  type="tel"
                  class="w-full px-4 py-2 bg-input border border-border rounded-lg text-foreground focus:outline-none focus:ring-2 focus:ring-ring"
                />
              </div>
              
              <button
                type="submit"
                disabled={isLoading || !userName.trim() || !userEmail.trim()}
                class="w-full px-6 py-3 bg-primary text-primary-foreground rounded-lg hover:bg-primary/90 disabled:opacity-50 transition-colors font-medium"
              >
                {isLoading ? 'Posting Job...' : 'Post My Job'}
              </button>
            </form>
          </div>
        </div>
      {:else if currentStep === 'confirmation'}
        <div class="max-w-md mx-auto text-center fade-in">
          <div class="bg-card border border-border rounded-xl p-8">
            <div class="w-16 h-16 bg-green-500/10 rounded-full flex items-center justify-center mx-auto mb-4">
              <svg class="w-8 h-8 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
              </svg>
            </div>
            
            <h3 class="text-2xl font-bold text-foreground mb-4">Job Posted Successfully!</h3>
            <p class="text-muted-foreground mb-6">
              Job ID: <span class="font-mono text-foreground">{jobId}</span><br/>
              Contractors are already viewing your job and preparing bids...
            </p>
            
            <div class="flex items-center justify-center space-x-2 mb-6">
              <div class="animate-spin rounded-full h-4 w-4 border-b-2 border-primary"></div>
              <span class="text-sm text-muted-foreground">Waiting for bids...</span>
            </div>
          </div>
        </div>
      {:else if currentStep === 'bids'}
        <div class="max-w-3xl mx-auto fade-in">
          <div class="text-center mb-8">
            <h2 class="text-3xl font-bold text-foreground mb-2">Choose Your Contractor</h2>
            <p class="text-muted-foreground">You received {bids.length} bids for your job</p>
          </div>
          
          <div class="space-y-4">
            {#each bids as bid}
              <div class="bg-card border border-border rounded-xl p-6 hover:border-primary/50 transition-colors">
                <div class="flex items-start justify-between">
                  <div class="flex items-start space-x-4">
                    <img src={bid.avatar || "/placeholder.svg"} alt={bid.name} class="w-12 h-12 rounded-full object-cover" />
                    <div class="flex-1">
                      <h3 class="text-lg font-semibold text-foreground">{bid.name}</h3>
                      <div class="flex items-center space-x-2 mb-2">
                        <div class="flex items-center">
                          {#each Array(5) as _, i}
                            <svg class="w-4 h-4 {i < Math.floor(bid.rating) ? 'text-yellow-400' : 'text-gray-300'}" fill="currentColor" viewBox="0 0 20 20">
                              <path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z" />
                            </svg>
                          {/each}
                          <span class="text-sm text-muted-foreground ml-1">{bid.rating} ({bid.reviews} reviews)</span>
                        </div>
                      </div>
                      <p class="text-muted-foreground text-sm mb-3">{bid.description}</p>
                      <div class="flex items-center space-x-4 text-sm">
                        <span class="text-foreground">Timeline: <strong>{bid.timeline}</strong></span>
                      </div>
                    </div>
                  </div>
                  <div class="text-right">
                    <div class="text-2xl font-bold text-foreground mb-2">${bid.price}</div>
                    <button
                      onclick={() => selectBid(bid)}
                      class="px-4 py-2 bg-primary text-primary-foreground rounded-lg hover:bg-primary/90 transition-colors"
                    >
                      Select
                    </button>
                  </div>
                </div>
              </div>
            {/each}
          </div>
        </div>
      {:else if currentStep === 'contract'}
        <div class="max-w-2xl mx-auto fade-in">
          <div class="bg-card border border-border rounded-xl p-6">
            <h2 class="text-2xl font-bold text-foreground mb-6 text-center">Contract Agreement</h2>
            
            <div class="bg-muted/50 rounded-lg p-4 mb-6">
              <h3 class="font-semibold text-foreground mb-3">Job Details</h3>
              <div class="space-y-2 text-sm">
                <div><strong>Job:</strong> {jobDescription}</div>
                <div><strong>Contractor:</strong> {selectedBid.name}</div>
                <div><strong>Price:</strong> ${selectedBid.price}</div>
                <div><strong>Timeline:</strong> {selectedBid.timeline}</div>
                <div><strong>Job ID:</strong> {jobId}</div>
              </div>
            </div>
            
            <div class="bg-muted/30 rounded-lg p-4 mb-6 text-sm text-muted-foreground">
              <h4 class="font-semibold text-foreground mb-2">Terms & Conditions</h4>
              <ul class="space-y-1">
                <li>• Payment will be held in escrow until job completion</li>
                <li>• Contractor is licensed and insured</li>
                <li>• Work must be completed within agreed timeline</li>
                <li>• Quality guarantee for 30 days after completion</li>
                <li>• Dispute resolution available through QuickHire</li>
              </ul>
            </div>
            
            <div class="flex items-center space-x-3 mb-6">
              <input type="checkbox" id="agreeTerms" class="w-4 h-4 text-primary bg-input border-border rounded focus:ring-primary" />
              <label for="agreeTerms" class="text-sm text-foreground">
                I agree to the terms and conditions and authorize the contractor to begin work
              </label>
            </div>
            
            <button
              onclick={signContract}
              disabled={isLoading}
              class="w-full px-6 py-3 bg-primary text-primary-foreground rounded-lg hover:bg-primary/90 disabled:opacity-50 transition-colors font-medium"
            >
              {isLoading ? 'Processing...' : 'Sign Contract & Proceed to Payment'}
            </button>
          </div>
        </div>
      {:else if currentStep === 'payment'}
        <div class="max-w-md mx-auto fade-in">
          <div class="bg-card border border-border rounded-xl p-6">
            <h2 class="text-2xl font-bold text-foreground mb-6 text-center">Secure Payment</h2>
            
            <div class="bg-green-500/10 border border-green-500/20 rounded-lg p-4 mb-6">
              <div class="flex items-center space-x-2 mb-2">
                <svg class="w-5 h-5 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                </svg>
                <span class="text-green-500 font-semibold">Escrow Protection</span>
              </div>
              <p class="text-sm text-green-600">Your payment is held securely until work is completed to your satisfaction.</p>
            </div>
            
            <div class="mb-6">
              <div class="flex justify-between items-center mb-2">
                <span class="text-foreground">Job Total:</span>
                <span class="text-2xl font-bold text-foreground">${paymentAmount}</span>
              </div>
              <div class="text-sm text-muted-foreground">
                Includes QuickHire service fee and payment processing
              </div>
            </div>
            
            <form onsubmit={processPayment} class="space-y-4">
              <div>
                <label for="paymentMethod" class="block text-sm font-medium text-foreground mb-2">Payment Method</label>
                <select id="paymentMethod" bind:value={paymentMethod} class="w-full px-4 py-2 bg-input border border-border rounded-lg text-foreground focus:outline-none focus:ring-2 focus:ring-ring">
                  <option value="card">Credit/Debit Card</option>
                  <option value="bank">Bank Transfer</option>
                  <option value="paypal">PayPal</option>
                </select>
              </div>
              
              {#if paymentMethod === 'card'}
                <div class="space-y-3">
                  <div>
                    <label for="cardNumber" class="sr-only">Card Number</label>
                    <input id="cardNumber" type="text" placeholder="Card Number" class="w-full px-4 py-2 bg-input border border-border rounded-lg text-foreground placeholder-muted-foreground focus:outline-none focus:ring-2 focus:ring-ring" />
                  </div>
                  <div class="grid grid-cols-2 gap-3">
                    <div>
                      <label for="cardExpiry" class="sr-only">Expiry Date</label>
                      <input id="cardExpiry" type="text" placeholder="MM/YY" class="px-4 py-2 bg-input border border-border rounded-lg text-foreground placeholder-muted-foreground focus:outline-none focus:ring-2 focus:ring-ring" />
                    </div>
                    <div>
                      <label for="cardCvc" class="sr-only">CVC</label>
                      <input id="cardCvc" type="text" placeholder="CVC" class="px-4 py-2 bg-input border border-border rounded-lg text-foreground placeholder-muted-foreground focus:outline-none focus:ring-2 focus:ring-ring" />
                    </div>
                  </div>
                  <div>
                    <label for="cardholderName" class="sr-only">Cardholder Name</label>
                    <input id="cardholderName" type="text" placeholder="Cardholder Name" class="w-full px-4 py-2 bg-input border border-border rounded-lg text-foreground placeholder-muted-foreground focus:outline-none focus:ring-2 focus:ring-ring" />
                  </div>
                </div>
              {/if}
              
              <button
                type="submit"
                disabled={isLoading}
                class="w-full px-6 py-3 bg-primary text-primary-foreground rounded-lg hover:bg-primary/90 disabled:opacity-50 transition-colors font-medium"
              >
                {isLoading ? 'Processing Payment...' : `Pay $${paymentAmount}`}
              </button>
            </form>
            
            <div class="mt-4 text-center text-xs text-muted-foreground">
              <div class="flex items-center justify-center space-x-2">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                </svg>
                <span>256-bit SSL encryption</span>
              </div>
            </div>
          </div>
        </div>
      {/if}
    </div>
  
    <footer class="border-t border-border/50 mt-20">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div class="flex flex-col md:flex-row justify-between items-center">
          <div class="text-sm text-muted-foreground mb-4 md:mb-0">
            © 2025 QuickHire. Connecting you with trusted contractors.
          </div>
          <div class="flex space-x-6 text-sm text-muted-foreground">
            <a href="/privacy" class="hover:text-foreground transition-colors">Privacy</a>
            <a href="/terms" class="hover:text-foreground transition-colors">Terms</a>
            <a href="/support" class="hover:text-foreground transition-colors">Support</a>
          </div>
        </div>
      </div>
    </footer>
  </main>
  