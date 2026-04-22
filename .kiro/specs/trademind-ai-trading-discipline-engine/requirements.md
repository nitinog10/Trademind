# Requirements Document

## Introduction

TradeMind AI is an AI-powered Trading Discipline Engine designed to help retail investors avoid bad trades by analyzing risk, detecting emotional trading patterns, and enforcing discipline through intelligent trade rejection. Unlike existing trading applications that focus on helping users find trades faster, TradeMind AI helps users trade smarter by preventing poor decisions, managing risk exposure, and building disciplined trading habits.

The system uses Google Gemini AI to analyze trade proposals before execution, maintains a local SQLite database for privacy, and provides real-time feedback on trading decisions. It targets the growing retail investor market (18+ crore investors in India) who often underperform due to decision confusion rather than lack of information.

## Glossary

- **TradeMind_System**: The complete AI-powered trading discipline engine
- **Risk_Profile_Engine**: Component that creates personalized trading frameworks based on user capital and risk tolerance
- **Trade_Analyzer**: AI component that evaluates proposed trades and returns structured decisions
- **Emotional_Detector**: Component that identifies emotional trading patterns like overtrading and revenge trading
- **Trade_Rejection_System**: Component that blocks risky trades based on AI analysis and risk limits
- **Kill_Switch**: Middleware that prevents all trading when daily loss limits are exceeded
- **Discipline_Dashboard**: UI component displaying capital preserved, risk exposure, and trading metrics
- **Learning_Engine**: Component that provides post-trade analysis and pattern detection
- **User_Risk_Profile**: Stored configuration containing max daily loss, max trades per day, and risk/reward ratios
- **Trade_Log**: Database record of approved and rejected trades with timestamps and reasoning
- **Gemini_Reasoning_Engine**: AI service wrapper that acts as a strict risk manager
- **Capital_Preserved**: Calculated metric showing money saved by rejecting bad trades
- **Loss_Prevention_Mode**: State where trading is locked after continuous losses
- **Discipline_Score**: Metric tracking user's adherence to trading discipline
- **Trade_Proposal**: User-submitted trade details including stock name, entry price, stop loss, and reasoning

## Requirements

### Requirement 1: Risk Profile Configuration

**User Story:** As a retail investor, I want to configure my risk profile with capital amount and risk tolerance, so that the system can create personalized trading limits that protect my capital.

#### Acceptance Criteria

1. THE Risk_Profile_Engine SHALL accept capital amount as a positive decimal value
2. THE Risk_Profile_Engine SHALL accept risk tolerance as one of three values: conservative, moderate, or aggressive
3. THE Risk_Profile_Engine SHALL accept trading style as one of three values: intraday, swing, or long-term
4. WHEN a User_Risk_Profile is created, THE Risk_Profile_Engine SHALL calculate maximum daily loss limit based on capital and risk tolerance
5. WHEN a User_Risk_Profile is created, THE Risk_Profile_Engine SHALL calculate maximum trades per day based on trading style
6. WHEN a User_Risk_Profile is created, THE Risk_Profile_Engine SHALL calculate minimum risk-reward ratio based on risk tolerance
7. THE Risk_Profile_Engine SHALL store the User_Risk_Profile in the local SQLite database
8. WHEN capital amount is less than or equal to zero, THE Risk_Profile_Engine SHALL return a validation error
9. THE Risk_Profile_Engine SHALL allow users to update their User_Risk_Profile at any time

### Requirement 2: Trade Proposal Submission

**User Story:** As a retail investor, I want to submit trade proposals with stock details and reasoning, so that the AI can analyze whether the trade aligns with my risk profile.

#### Acceptance Criteria

1. THE TradeMind_System SHALL accept Trade_Proposal with stock name, entry price, stop loss, target price, quantity, and reasoning text
2. WHEN entry price is less than or equal to zero, THE TradeMind_System SHALL return a validation error
3. WHEN stop loss is greater than or equal to entry price for a long position, THE TradeMind_System SHALL return a validation error
4. WHEN target price is less than or equal to entry price for a long position, THE TradeMind_System SHALL return a validation error
5. WHEN quantity is less than or equal to zero, THE TradeMind_System SHALL return a validation error
6. WHEN reasoning text is empty or less than 10 characters, THE TradeMind_System SHALL return a validation error
7. THE TradeMind_System SHALL calculate total capital at risk as quantity multiplied by the difference between entry price and stop loss
8. THE TradeMind_System SHALL calculate risk-reward ratio as the ratio of potential profit to potential loss

### Requirement 3: AI Trade Analysis

**User Story:** As a retail investor, I want the AI to analyze my trade proposal and provide a structured decision, so that I can understand whether the trade is worth taking.

#### Acceptance Criteria

1. WHEN a valid Trade_Proposal is submitted, THE Trade_Analyzer SHALL send the proposal to the Gemini_Reasoning_Engine
2. THE Gemini_Reasoning_Engine SHALL analyze the Trade_Proposal using the User_Risk_Profile as context
3. THE Gemini_Reasoning_Engine SHALL return a structured decision with one of three values: REJECT, CAUTION, or APPROVE
4. THE Gemini_Reasoning_Engine SHALL return a reason explaining the decision in natural language
5. THE Gemini_Reasoning_Engine SHALL return a risk percentage calculated from the Trade_Proposal
6. THE Gemini_Reasoning_Engine SHALL return a discipline score impact ranging from -10 to +10
7. THE Gemini_Reasoning_Engine SHALL return specific warnings as an array of strings when risks are identified
8. WHEN the Gemini_Reasoning_Engine returns REJECT, THE Trade_Rejection_System SHALL prevent the trade from being executed
9. WHEN the Gemini_Reasoning_Engine returns CAUTION, THE TradeMind_System SHALL display warnings but allow the user to proceed
10. WHEN the Gemini_Reasoning_Engine returns APPROVE, THE TradeMind_System SHALL allow the trade to be executed

### Requirement 4: Trade Logging and History

**User Story:** As a retail investor, I want all my approved and rejected trades to be logged, so that I can analyze my trading patterns and impulse rate over time.

#### Acceptance Criteria

1. WHEN a Trade_Proposal receives a decision from the Trade_Analyzer, THE TradeMind_System SHALL create a Trade_Log entry
2. THE Trade_Log SHALL store the stock name, entry price, stop loss, target price, quantity, and reasoning
3. THE Trade_Log SHALL store the AI decision as REJECT, CAUTION, or APPROVE
4. THE Trade_Log SHALL store the AI reason and warnings
5. THE Trade_Log SHALL store the timestamp of the trade proposal
6. THE Trade_Log SHALL store whether the user proceeded with the trade after receiving CAUTION
7. THE Trade_Log SHALL store the calculated risk percentage and risk-reward ratio
8. THE TradeMind_System SHALL retrieve the last 50 Trade_Log entries for pattern analysis
9. THE TradeMind_System SHALL calculate impulse rate as the percentage of rejected trades to total trade proposals

### Requirement 5: Emotional Trading Detection

**User Story:** As a retail investor, I want the system to detect when I'm trading emotionally, so that I can avoid making impulsive decisions that harm my capital.

#### Acceptance Criteria

1. WHEN a user submits more trades than the maximum trades per day from their User_Risk_Profile, THE Emotional_Detector SHALL flag overtrading
2. WHEN a user submits a Trade_Proposal within 30 minutes of a rejected trade, THE Emotional_Detector SHALL flag potential revenge trading
3. WHEN a user has three or more consecutive rejected trades in the last 50 Trade_Log entries, THE Emotional_Detector SHALL flag a negative pattern
4. WHEN the Emotional_Detector flags overtrading, THE TradeMind_System SHALL display an alert to the user
5. WHEN the Emotional_Detector flags revenge trading, THE Trade_Analyzer SHALL include this context in the Gemini_Reasoning_Engine analysis
6. WHEN the Emotional_Detector flags a negative pattern, THE TradeMind_System SHALL recommend taking a break from trading
7. THE Emotional_Detector SHALL use the last 50 Trade_Log entries as context for pattern detection

### Requirement 6: Kill Switch and Loss Prevention

**User Story:** As a retail investor, I want the system to automatically stop me from trading when I hit my daily loss limit, so that I don't lose more money than I can afford.

#### Acceptance Criteria

1. THE Kill_Switch SHALL calculate total daily losses from Trade_Log entries with timestamps from the current day
2. WHEN total daily losses exceed the maximum daily loss limit from the User_Risk_Profile, THE Kill_Switch SHALL activate
3. WHEN the Kill_Switch is active, THE Trade_Rejection_System SHALL block all new Trade_Proposal submissions
4. WHEN the Kill_Switch is active, THE TradeMind_System SHALL display a message explaining that trading is locked until the next day
5. WHEN the Kill_Switch is active, THE TradeMind_System SHALL display the total daily losses and the maximum daily loss limit
6. WHEN a new day begins, THE Kill_Switch SHALL automatically deactivate
7. WHEN a user has three or more consecutive losing trades, THE TradeMind_System SHALL activate Loss_Prevention_Mode
8. WHEN Loss_Prevention_Mode is active, THE Trade_Analyzer SHALL apply stricter analysis criteria to Trade_Proposals

### Requirement 7: Discipline Dashboard Metrics

**User Story:** As a retail investor, I want to see a dashboard showing my trading discipline metrics, so that I can understand how much capital I've preserved and my overall risk exposure.

#### Acceptance Criteria

1. THE Discipline_Dashboard SHALL calculate Capital_Preserved as the sum of potential losses from all rejected trades
2. THE Discipline_Dashboard SHALL display total Capital_Preserved as a monetary value
3. THE Discipline_Dashboard SHALL calculate total risk exposure as the sum of capital at risk from all approved trades that are still open
4. THE Discipline_Dashboard SHALL display total risk exposure as a monetary value and percentage of total capital
5. THE Discipline_Dashboard SHALL calculate sector concentration by grouping trades by stock sector
6. THE Discipline_Dashboard SHALL display sector concentration as percentages
7. THE Discipline_Dashboard SHALL calculate worst-case scenario as the total potential loss if all open trades hit their stop losses
8. THE Discipline_Dashboard SHALL display worst-case scenario as a monetary value and percentage of total capital
9. THE Discipline_Dashboard SHALL display the current Discipline_Score
10. THE Discipline_Dashboard SHALL display impulse rate as a percentage

### Requirement 8: Learning and Post-Trade Analysis

**User Story:** As a retail investor, I want the AI to explain what went right or wrong after every trade, so that I can learn from my decisions and improve my trading discipline.

#### Acceptance Criteria

1. WHEN a trade is completed, THE Learning_Engine SHALL analyze the trade outcome against the original Trade_Proposal
2. THE Learning_Engine SHALL compare the actual exit price with the target price and stop loss
3. THE Learning_Engine SHALL send the trade outcome and the last 50 Trade_Log entries to the Gemini_Reasoning_Engine for analysis
4. THE Gemini_Reasoning_Engine SHALL return a learning summary explaining what went right or wrong
5. THE Gemini_Reasoning_Engine SHALL identify whether the trade followed the original plan
6. THE Gemini_Reasoning_Engine SHALL detect revenge trading patterns by analyzing trade timing and reasoning
7. THE Learning_Engine SHALL store the learning summary in the Trade_Log
8. THE TradeMind_System SHALL display the learning summary to the user after trade completion

### Requirement 9: Gemini AI Integration

**User Story:** As a developer, I want to integrate Google Gemini API for trade reasoning and emotional analysis, so that the system can provide intelligent trade decisions.

#### Acceptance Criteria

1. THE Gemini_Reasoning_Engine SHALL use the @google/generative-ai library to connect to Google Gemini API
2. THE Gemini_Reasoning_Engine SHALL use a system prompt that acts as a strict risk manager
3. THE Gemini_Reasoning_Engine SHALL send Trade_Proposal details and User_Risk_Profile as context
4. THE Gemini_Reasoning_Engine SHALL send the last 50 Trade_Log entries for pattern detection
5. THE Gemini_Reasoning_Engine SHALL request structured JSON output with decision, reason, risk_percentage, discipline_score_impact, and warnings fields
6. WHEN the Gemini API returns an error, THE Gemini_Reasoning_Engine SHALL return a default REJECT decision with an error message
7. WHEN the Gemini API response is not valid JSON, THE Gemini_Reasoning_Engine SHALL return a default REJECT decision with a parsing error message
8. THE Gemini_Reasoning_Engine SHALL use a timeout of 10 seconds for API requests

### Requirement 10: Database Schema and Persistence

**User Story:** As a developer, I want to use SQLite with Prisma ORM for local data storage, so that user trading data remains private and the system responds instantly.

#### Acceptance Criteria

1. THE TradeMind_System SHALL use SQLite as the database engine
2. THE TradeMind_System SHALL use Prisma ORM for database operations
3. THE TradeMind_System SHALL define a User_Risk_Profile schema with fields for capital, risk_tolerance, trading_style, max_daily_loss, max_trades_per_day, and min_risk_reward_ratio
4. THE TradeMind_System SHALL define a Trade_Log schema with fields for stock_name, entry_price, stop_loss, target_price, quantity, reasoning, ai_decision, ai_reason, ai_warnings, timestamp, risk_percentage, risk_reward_ratio, user_proceeded, and learning_summary
5. THE TradeMind_System SHALL store all data in a local SQLite file
6. THE TradeMind_System SHALL create database indexes on timestamp fields for efficient querying
7. THE TradeMind_System SHALL support database migrations through Prisma Migrate

### Requirement 11: User Interface Components

**User Story:** As a retail investor, I want a clean and intuitive interface built with Next.js and Shadcn UI, so that I can easily submit trades and view my discipline metrics.

#### Acceptance Criteria

1. THE TradeMind_System SHALL use Next.js 14 with App Router for the frontend framework
2. THE TradeMind_System SHALL use Tailwind CSS for styling
3. THE TradeMind_System SHALL use Shadcn UI components for consistent design
4. THE TradeMind_System SHALL provide a trade input form with fields for stock name, entry price, stop loss, target price, quantity, and reasoning
5. THE TradeMind_System SHALL use React Hook Form for form state management
6. THE TradeMind_System SHALL use Zod for input validation schemas
7. WHEN form validation fails, THE TradeMind_System SHALL display inline error messages
8. THE TradeMind_System SHALL display AI analysis results with color-coded decisions: red for REJECT, yellow for CAUTION, green for APPROVE
9. THE TradeMind_System SHALL provide a risk profile configuration page
10. THE TradeMind_System SHALL provide a trade history page showing both approved and rejected trades
11. THE TradeMind_System SHALL provide the Discipline_Dashboard as the main landing page

### Requirement 12: Server Actions and API Layer

**User Story:** As a developer, I want to use Next.js Server Actions for backend logic, so that the system can securely process trades and interact with the database.

#### Acceptance Criteria

1. THE TradeMind_System SHALL implement a Server Action for submitting Trade_Proposals
2. THE TradeMind_System SHALL implement a Server Action for creating and updating User_Risk_Profile
3. THE TradeMind_System SHALL implement a Server Action for retrieving Trade_Log history
4. THE TradeMind_System SHALL implement a Server Action for calculating Discipline_Dashboard metrics
5. THE TradeMind_System SHALL implement a Server Action for completing trades and triggering Learning_Engine analysis
6. WHEN a Server Action encounters a database error, THE TradeMind_System SHALL return a structured error response
7. THE TradeMind_System SHALL validate all inputs in Server Actions using Zod schemas
8. THE TradeMind_System SHALL implement the Kill_Switch as middleware in the trade submission Server Action
